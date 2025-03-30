---
layout: default
---

## The classic ** Mr. Robot** challenge where we have to find 3 hidden keys on the machine.

Preparation: add the IP of the machine to `/etc/hosts` to be able to use the domain name: `robot.thm`

### Nmap

Initial scan shows that ports 22, 80, and 443 are open, so I guess we’re dealing with a web server hack

![image](https://github.com/user-attachments/assets/e5a0707e-1c61-4775-95f9-8b12818b250e)


A version scan on the HTTP ports just shows that we’re dealing with an Apache server:

![image](https://github.com/user-attachments/assets/80d0a0bd-686c-435f-9a61-99d6201d6580)


### Port 80 Web Enumeration:

A simple curl request returns a comment ‘YOU ARE NOT ALONE’

![image](https://github.com/user-attachments/assets/081906a6-060e-49c5-9cc8-925bde607192)


Main page has a mock command line with a few listed prompts to try:

![image](https://github.com/user-attachments/assets/c414a366-d6b4-45ba-a030-6fc2a6de7734)

1. `prepare` -> Intro video
2. `fsociety` -> "Are you ready to joing fsociety?"
3. `inform` -> Mr. Robot's comments on corruption and immorality news
    - "Counterfeit heroes"	- card usage ("infection") - star ("frauds")
    - Comments on American patriot, executive, capitalist, and businessman
4. `wakeup` -> Video of people arguing?
5. `join` -> Enter your email address and "I'll be in touch"

### Directory Busting with Gobuster

Gobuster scan returned quite a few interesting directories, and we now can assume that this is a Wordpress site and it uses MySQL and PHP

![image](https://github.com/user-attachments/assets/2d202a77-f88c-43e1-9bf5-074b576f84f7)

![image](https://github.com/user-attachments/assets/30d9c4e2-7395-4dac-9a9d-46042e57d360)


`/robots`    Returns two files that are held on the server, the latter being the first key:

![image](https://github.com/user-attachments/assets/ae3baee6-c4c4-42cc-b98d-e4a15fa2d6de)

![image](https://github.com/user-attachments/assets/c5a128a1-2a74-4b98-b7b9-ba8fd0312fb5)


`feed`    Contains a file that has some configuration info

`wp-login`   This looks like the default Wordpress login screen

`license`   Displays the message “Since when did you become a script kiddy?" Haha. Well, that’s not going to throw me off…..and look, we have base64 data:

![image](https://github.com/user-attachments/assets/a0a3d78b-11c6-4b92-9918-150b9461366e)


Decoding the Base64 data, it looks like we have some credentials:

![image](https://github.com/user-attachments/assets/4db0f25e-e5de-498e-8271-f280f6b7b887)


### Exploitation

And we successfully log into `/wp-login` with those credentials….it also looks like Elliot is the admin.

![image](https://github.com/user-attachments/assets/865380a6-2a4d-4ab9-9109-888941882b70)


Main page reveals WP version 4.3.1 and Theme TwentyFifteen

- After research, discovered that this WP version is vulnerable to an XSS attack because it fails to properly sanitize user-supplied input.

CVE-2016-1564: "XSS vulnerabilities in wp-includes/class-wp-theme.php...inject arbitrary web script or HTML via a stylesheet name or template name to wp-admin/customize.php

So, we can likely inject a reverse shell PHP file to one of the templates or stylesheets on admin page. I chose to do it in the 404.php page:

![image](https://github.com/user-attachments/assets/28f3f447-2dd8-4a5a-829a-852d3132b9fe)


Started a Netcat listener on my machine and then accessed where the 404.php would be shown.

![image](https://github.com/user-attachments/assets/8c64bcc6-5886-4409-9bf7-1c9d07d18ec4)


The page was empty, but it sparked a connection from the target to my listener, and now we have a shell on their server:

![image](https://github.com/user-attachments/assets/4f09b97c-478f-4128-ad1d-84b8bc98b200)


### Lateral Movement

The `/etc/passwd` file returned three non-root users on the machine: `mysql`   `varnish`   `robot`

![image](https://github.com/user-attachments/assets/8beec3a3-2e8f-480f-a177-828c51c20907)


Did a search for the key file on the machine and got a permission denied

![image](https://github.com/user-attachments/assets/14a9de29-4e64-40b0-8464-610694d09e76)


But a quick look into his home directory and we find that his credentials are listed in a not-so-secure file, with the password being an MD5 hash: 

![image](https://github.com/user-attachments/assets/c300cf27-fefa-4fc9-87cc-01ab16026b93)


A simple Hashcat crack returns none other than the alphabet as his password:

![image](https://github.com/user-attachments/assets/c66597d9-d7bd-4871-8e95-00a1b498ca22)


So, now we just have to switch to the `robot` user and get the next key…..but first, we have to spawn an actual terminal, so I did that beforehand:

![image](https://github.com/user-attachments/assets/16e3779e-e280-4aed-8ce5-10e75c2f2672)


### Privilege Escalation

I’m definitely assuming the final key will be in the root directory or at least accessible only by root, so it looks like we’ll have to ‘root’ the machine

Did some usual checks, such as any cronjobs or commands we can run as sudo, but nothing.

Finally, I looked for binaries that have SUID privileges and an interesting one turned up: **Nmap**

![image](https://github.com/user-attachments/assets/bb42545e-e090-4e7e-adc8-87a8680a7c91)


A quick look on GTFObins and we see that getting an interactive system shell is as simple as this:

![image](https://github.com/user-attachments/assets/6f698574-05ce-4a48-89e9-5e3197740409)


And with that, we can get our final key:

![image](https://github.com/user-attachments/assets/c88b360e-e160-4df0-a36c-8520c04f9c5e)


Mission accomplished!

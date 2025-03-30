---
layout: default
---

# OverTheWire's Bandit Challenge
### Informal writeup with a collection of screenshots and some notes as I made my way through the Bandit challenge from OverTheWire, which I revisited after investing some more time in learning scripting.

**Level 1: the password is stored in a file called "readme"** 

SSH into the game as bandit 0 on port 2220

![image](https://github.com/user-attachments/assets/9846544d-2e7c-4aa0-bf47-4cce10e19efc)

Cat the ***readme*** file

![image](https://github.com/user-attachments/assets/30b00cb0-8f14-4591-8388-5cebb7947d3f)

**Level 2: the password is stored in a file called `-`**

SSH as bandit1 and cat the `-` file

![image](https://github.com/user-attachments/assets/490665c5-cc54-46de-8f13-aa99606960a7)

- I used **`find`** to see how to write this out → in short, you apparently prefix with `*./*` to cat out files beginning with symbols.

**Level 3: the password is in a file called "spaces in this filename"** 

SSH as bandit2 and cat the filename with the spaces:

![image](https://github.com/user-attachments/assets/dcaa4526-74f3-427a-b57c-6e46a54c70bb)

**Level 4: the password is in a hiddne file called "inhere"**

SSH as bandit3, CD into directory, and cat the hidden file

![image](https://github.com/user-attachments/assets/9e152831-b13e-4231-a9f7-2a455a517c85)

**Level 5: the password is stored in the only human-readable file in the "inhere" directory**

SSH as bandit4 and retrieve the only human-readable text from the list of files:

![image](https://github.com/user-attachments/assets/47f37c85-da3b-481d-848d-79cdb72eb5de)

Since the files were each very small, I simply catted all of them out to save time **(not pretty, but it works).

**Level 6: the password is stored in "inhere" directory, is human-readable, is 1033 bytes, and is not executable** 

![image](https://github.com/user-attachments/assets/fe364b78-c652-4bb4-8590-20a92322902e)

Managed to find the right file matching only two of the conditions:

- `-size 1033c` limits the results to files with 1033 bytes
- `\! -executable` shows only files that are NOT executable, with the `\!` acting as a negation.

**Level 7: the password is stored somewhere on the server, is owned by user bandit7 and group bandit6, and is 33 bytes**

![image](https://github.com/user-attachments/assets/a7fd6dfd-d5b8-4942-9965-05936d7e7936)

- `2>/dev/null` avoids the “Permission Denied” errors by dumping such output into garbage space

**Level 8: the password is stored in the file "data.txt" next to the word "millionth"**

![image](https://github.com/user-attachments/assets/e42efa64-3c1a-4247-b0c2-e0ed6ceffb84)

**Level 9: the password is stored in the file "data.txt" and is the only line that occurs just once**

![image](https://github.com/user-attachments/assets/b86eb5de-93cc-46a5-8d26-757ad83d0d85)

We naturally have to `sort` the file first for it to check for duplicates, and then, only return the unique lines via `uniq -u`)

**Level 10: the password is stored "file.txt", is human-readable, and is preceded by several "=" signs** 

![image](https://github.com/user-attachments/assets/87be6146-8c89-435f-8c72-76d039a6570c)

- `strings` filters for just the human-readable text
- `grep` can filter for a specific pattern, with the wildcard (`*`) representing ‘anything’

**Level 11: the passwords is in "data.txt", which is base64-encoded**

Simply decode the base64-encoded data:

![image](https://github.com/user-attachments/assets/7917a8ac-cea5-4778-812f-d34249d11fef)

**Level 12: the passwords is in "data.txt", where all letters are rotated by 13 positions**

Classic case of Rot13, so we just need to rotate the characters back:

![image](https://github.com/user-attachments/assets/f772ba9b-bbad-4474-9d37-59934ab97456)

`tr` can be used to ‘translate’ (or ‘rotate’) the letters by 13 characters.

Logic:
- We essentially swap the characters, starting with the letter ‘A’ (the 1st letter in the alphabet) and the letter ‘N’ (the 14th letter in the alphabet, or 13 letters ahead of ‘A’).
- Both lower and uppercase letters have to be swapped separately.

**Level 13:  the password is stored in "data.txt", which is a hexdump of a file that's bene repeatedly compressed**

Make a temporary directory to work with the file

![image](https://github.com/user-attachments/assets/827b7ea2-2365-443e-b258-05d49a9c783b)

Revert the hexdump to its original form:

![image](https://github.com/user-attachments/assets/2fb034f0-3d7a-46fd-b4b0-2890c1a5dcd9)

Before it was converted to a hexdump, the file was a Gzip called “data2.bin”

It took many, many iterations to decompress the file, and it involved the following process:

1. Display what type of file it is with `file`
2. If in Gzip format: `mv data1 data2.gz` → `gzip -d data2.gz`
3. If in Bzip format: `bzip2 -d data2`
4. If in Tar format: `tar -xf data2`

![image](https://github.com/user-attachments/assets/86adc5b9-87be-45db-8eb8-531c5cbaaab3)

At data9, we FINALLY have ASCII text, which contains the password.
    
![image](https://github.com/user-attachments/assets/39bdfc18-b6ed-47db-8074-126c74a62365)

**Level 14: use the SSH key to log in as bandit14**

This is a classic RSA key:

![image](https://github.com/user-attachments/assets/68745977-0a45-4d0e-a4f8-ae1dec8707d8)

Copied the key into a file **bandit14.rsa,** modified the permissions to be able to use it, then SSHed as bandit14 with the key:

![image](https://github.com/user-attachments/assets/808b8755-6383-4f34-bccb-f656a6a0e17d)

**Level 15: submit bandit14’s password to port 30000 to retrieve the next password** 

The previous level mentioned where the password was, so we just connect to the port via Netcat and send over the contents of the password file through a pipe (`|`) and get its reply:

![image](https://github.com/user-attachments/assets/1a41452f-e70c-4b8a-81ce-607ba02139f3)

**Level 16: submit bandit15's password to port 30001 using SSL/TLS encryption** 

We have to use `openssl` to connect securely (with an encrypted session) to that port:

![image](https://github.com/user-attachments/assets/0bdee8b9-e038-43ef-94b1-76baefde832d)

Simply submitted bandit15’s password and it returned the next one:

![image](https://github.com/user-attachments/assets/0845ae1b-5689-4110-bd26-32b189ab591f)

**Level 17: find a listening port that speaks SSL/TLS and submit bandit16's password to it.**

Easy method would be to run Nmap locally to find open ports in the range given:

![image](https://github.com/user-attachments/assets/8e750ff8-5d48-445d-b190-e19efefa5500)

The more challenging method is to write out your own script, which is useful for when you don’t have port scanning tools such as Nmap available on the local system.

So, I made a temp directory to write out the script:

![image](https://github.com/user-attachments/assets/cde5bab3-e763-44e8-b423-cd240b23b2c8)

**portscanner.py**

```python
import socket

# Variables: IP to range of ports to scan
ip = "127.0.0.1"
port_range = range(31000, 32001)

openPorts = [] # empty list to add matches to

# Iterate over each port
for port in port_range:

        # Create a socket and attempt to connect to the port
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        result = sock.connect_ex((ip, port))

        # If the connection is successful, append to the list:
        if result == 0:
                openPorts.append(f"Port {port} is open")

        # Close the socket (connection)
        sock.close()

# Print out the open ports
print("\n".join(openPorts))
```

Then, changed the permissions on the script and ran it:

![image](https://github.com/user-attachments/assets/ef092a1a-0955-41f7-82b5-9792b5951f8b)

_Open Ports: 31046, 31518, 31691, 31790, 31960_

Since there are only 5 ports open, I just tested each of them manually to see if they successfully connected with openssl:

![image](https://github.com/user-attachments/assets/a9946803-0905-4a2f-ade9-6a54b9961ca6)

Both 31518 and 31790 accept input, but there’s a KEYUPDATE generated, so I had to do some research into this error and ended up finding a parameter that can bypass it -> `-ign_eof`:

![image](https://github.com/user-attachments/assets/2cd23dc0-db5a-420e-baf0-eb29c498494e)

Port 31790 accepted the password and returned a private key

![image](https://github.com/user-attachments/assets/c4984d4c-4bc4-4d55-b15d-d3546854e4bd)

**Level 18: the password is the only line that’s different between the two given files**

Simply ran a `diff` against the two password files to find the change:

![image](https://github.com/user-attachments/assets/523816da-ec9f-4185-a74b-435f1a2ce6f1)

**Level 19: find a way to retrieve the password in "readme" while being immediately logged out of SSH upon loggin in"**

We should still be able to pass commands to the system through SSH even if we can’t use an interactive terminal, so since we know the name of the file we want, can just cat that:

![image](https://github.com/user-attachments/assets/592f093a-2e33-4ba2-98b2-9a093acca9c1)

**Level 20: use the binary file to get the next password**

`strings` returns a few interesting things:
    1. Line of text: “`Run a command as another user`.”
    2. `bandit20.c` → I’m wondering if the binary references this file.
    3. There’s also a location given `/usr/bin/env`, so maybe this is where the C file is?

![image](https://github.com/user-attachments/assets/7e3d37ec-d64c-44a4-9264-a926bda00385)

Since the file’s name references the bandit20 user, maybe all we need to do is run a cat command through the binary:

![image](https://github.com/user-attachments/assets/421a2ad0-d3b4-4978-ae38-52cfc224334a)

Yep…it was that simple. Note to self to always run the program BEFORE trying to reverse engineer it.

**Level 21: use the binary file to retrieve the password**

According to the level notes, the binary `suconnect` does the following: connects to 'localhost' on a specified port → reads a line of text from the connection → compares the line to bandit20’s password → if it’s a match, it transmits bandit21’s password.

Since the binary needs to retrieve the text for the comparison from a port connection, we probably need a way to store text in a port in the first place. I was not familiar with how to do this, so I had to do some research and discovered that we can actually do just that with Netcat:

![image](https://github.com/user-attachments/assets/5445d597-630b-4bfa-9749-b0cc3c56f6c9)

Essentially, we create a listening port 4444 and pass bandit20’s password to it, background the listener, and then have the binary access that listening port to retrieve the text.

And because the lines match, we do indeed get bandit21’s password.

**Level 22: use the program being run via cron to retrieve the password**

Found a script being run at reboot: `/usr/bin/cronjob_bandit22.sh`, which is a bash script that changes permissions on a temp file, then cats out bandit22’s password to that file.

![image](https://github.com/user-attachments/assets/b44ff3bb-caab-4f7d-a609-4ebab092e3a7)

Simply read the file referenced in that script and got the password.

**Level 23: another cronjob challenge**

Similar process to before, located a binary that copies the password files for a username provided by the `whoami` prompt over to a temp directory.

![image](https://github.com/user-attachments/assets/051a79eb-3544-4f58-8d6f-9aae90ade375)

The script isn’t writeable, but it can be run by bandit22 and I believe the key is to trick it into thinking we’re bandit23, as it runs the `whoami` command to get the username.

I tried making variables for both whoami and myname, but this didn’t work.

Through trial and error, I discovered that I could possibly make my own copy of the script with the same title as the existing one, change the PATH to start with the location of my own script, and then when the job runs next, it will run my script first (with root privileges of course).

So, I created a temp directory and prepended the PATH variable with that directory.

![image](https://github.com/user-attachments/assets/1317fb14-d98a-4293-8805-a13d3e0e3851)

I copied over the original script code to my own file with the same name as the original, changing the line that specifies the **myname** variable to the value “bandit23” instead of performing `whoami`:

And now, the temp directory is the first place that will be checked for the script instead of "/usr/bin".

Finally, the script details that the name of the file that will contain bandit23’s password is an MD5 hash, so I ran that line of code separately to get the exact value, then catted out the file:
    
![image](https://github.com/user-attachments/assets/aa8364c2-08ce-45d2-9fc2-f5fe4ee25f37)

I’m extremely proud of myself right now 😊

**Level 24: there's a time-based job that's running a command at regular intervals**

1. The script shows that everything within the directory /var/spool/bandit24/foo gets run periodically.
2. Made a simple reading script:
    
![image](https://github.com/user-attachments/assets/cbc08e97-f750-439a-b9f5-649e29efbd83)
    
3. Added executable privileges on my script.
4. Made a file called “bandit25” in the temp directory and changed its permissions, too.
5. Copied over the script to /var/spool/bandit24/foo and waited for it to run......and it didn’t
6. After a lot of digging, I found out it was because I needed to set the permissions of my temp directory, too
7. Ran it again and it worked:
    
![image](https://github.com/user-attachments/assets/7a0a1fa7-ffae-4a2d-8c57-9bf1d6645491)

**Level 25: bruteforce the 4-digit passcode for the port to get the password**

Wrote a simple bash script to send a range of 4-digit numbers and bandit24’s password to the Netcat port and send the output from the server to a text file:

```bash
#!/bin/bash

for num in {1000..9999}
do
    echo <password> $num
done | nc [localhost](http://localhost) 30002 > results.txt
```

Add executable permissions to the script and run it.

![image](https://github.com/user-attachments/assets/3e60f4c8-7585-4a3d-ac13-acccee10eb44)

Grep out the wrong answers to highlight the correct one

![image](https://github.com/user-attachments/assets/025871bd-ce48-4cfb-8597-46b7408025e1)

**Level 26: log in as bandit26 with the given key and break out of the shell**

1. Tried to SSH in with the key, but the shell immediately crashes.
2. Checked from bandit25's session to see what shell bandit26 is using, which is `showtext`.
3. Looking at the binary for this interesting shell, it looks like it displays a text file with the `more` command, then immediately exits.

![image](https://github.com/user-attachments/assets/5abf95fb-5ab7-460f-82d3-46e2af5bac35)

I did realize that the `more` command would play a role here, but it took a lot of digging to realize that it’s basically the entry point to gain a regular shell. And to prompt it to display, the terminal needs to be shrunk to a very small size:

![image](https://github.com/user-attachments/assets/751441b8-e4ad-4c00-93fe-5625323aa082)

Once this is triggered, `V` can be pressed to actually open up Vim and return to normal windows size, after which we can change the shell being used by bandit26 to a simple Bash via `:set shell=/bin/bash`, switch to it, and finally cat out bandit26’s password:
    
![image](https://github.com/user-attachments/assets/d7aa1b38-4658-453e-8199-395e005fc212)

**Level 27: grab the password**

There is a simple binary in the current directory where you can run a command as bandit27, so I used that to grab the password (most of the work was done in the previous level):

![image](https://github.com/user-attachments/assets/70cd8689-dc6d-46f0-89e8-44b482d102ac)

**Level 28: clone the Git repository and find the next password**

Created a temp directory and cloned the specified repository there, then just catted the README file:
    
![image](https://github.com/user-attachments/assets/0027cc12-e062-4c2b-8544-471ec1e085a8)

**Level 29: clone the Git repository and find the next password**

1. Same first step as previous level, but for bandit28’s repo.
2. The "README" file contains notes for the next level, including a placeholder for the password.
3. My first thought is that the password was written here before the current version of the file (because of the placeholder), so I’ll check the history:
    
![image](https://github.com/user-attachments/assets/47ed3c6a-01c0-4892-a40b-e96d976c9f0f)

And the previous version did indeed have the password.

**Level 30: clone the Git repository and find the next password**

1. Same first step as previous 2 levels, but for bandit29’s repo.
2. The README.md file says no passwords are in production, but I’ll double-check the history. Yes, it returns nothing.
3. `git log --all` → checked history of commits for all of the branches.
4. Found a commit with the note “add data needed for development”.
5. `git show <commit>` returned the original data for the password line, which is the actual password.

![image](https://github.com/user-attachments/assets/e3771a3b-dab0-4396-83c6-14f352d8b243)

**Level 31: clone the Git repository and find the next password**

1. Same first step as previous 3 levels, but for bandit30’s repo.
2. There is nothing in the logs this time, and there’s only one branch.
3. After trying various commands (referencing a categorized list I found online), `git tag` returned a tag called "secret".
4. `git show secret` returned what looks to be the password.
    
![image](https://github.com/user-attachments/assets/a7c08c9d-1d2f-4120-92fe-7d503e840407)

**Level 32: clone the Git repository and find the next password**

1. Same first step as previous 4 levels, but for bandit31’s repo.
2. The README file this time says that the task is to push a file called “key.txt” with the content “May I come in?” to the remote repository’s master branch.
    
![image](https://github.com/user-attachments/assets/f3849d0a-4dbb-404d-b3f4-7f65bb846f67)

3. Pushed the file to the remote repository, and the message displayed as confirmation of the commit’s receipt was the password for the next level.

![image](https://github.com/user-attachments/assets/8a53c624-68a8-46eb-a9ae-728c9deeeeea)

![image](https://github.com/user-attachments/assets/5d68e662-3dda-4344-ba4f-623a3fe37e12)

**Level 33: the only thing it says is this is an “escape” challenge** 

1. It says we are using a n “UPPERCASE SHELL”
2. Many commands are restricted, and it’s converting the commands to uppercase form.
3. I tried to see if environment variables could be assigned, but this didn’t work.
4. The hint given is the command `sh` which seems to be related to a Bash interpreter.
5. After reading the man page, I’m sure the key is in variables.
6. It ended up being as simple as switching to a regular shell via $0.

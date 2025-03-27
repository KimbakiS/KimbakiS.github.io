### Informal writeup, collection of notes as I made my way through the Bandit challenge from OverTheWire, which I revisited after investing some more time in learning scripting.

**Level 0:** ssh into the game as bandit 0 on port 2220

1. *bandit=bandit.labs.overthewire.org*
2. *echo $bandit*
3. *ssh bandit0@$bandit -p 2220*
4. Entered password *bandit0*
5. *cat readme* (contains the password)

**Level 1:** ssh into the game as bandit1 using the found password

1. *ssh bandit1@$bandit -p 2220*
2. Provide password from the readme file.
3. *cat ./-* (The filename was ‘-’ and I used *find* to see how to write this out → prefix with *./* for symbols)

**Level 2**: 

1. *cat “spaces in this filename”* (easy)

**Level 3:**

1. *cd inhere/*
2. *ls -la* (show hidden files)
3. *cat “… Hiding-From-You”* (display the contents of the hidden file

**Level 4:**

1. *cd inhere*
2. *ls* (shows many files)
3. *cat ./-file** (displays all of the contents of each file…not pretty, but it works)

**Level 5:** find a file that’s 1033 bytes, is not executable, and is human-readable.

1. I realized I was following the previous level’s directions and couldn’t find the file originally. 
2. *cd /inhere*
3. *find . -type f -size 1033c | xargs file | grep text* (display files with a size of 1033 bytes and run the *file* command via *xargs* on each of these lines, then grep out the results containing “text” (human-readable files)

**Level 6:** find a file that is owned by bandit7, is in group bandit6 and is 33 bytes

1. *find / -type f -user bandit7 -group bandit6 -size 33c 2>dev/null*
2. *cat /var/lib/dpkg/info/bandit7.password*

**Level 7:** find the password next to the word “millionth” in data.txt

1. grep millionth data.txt (seriously…)

**Level 8:** the password is the only line that occurs just once in data.txt

1. *cat data.txt | sort | uniq -u* (have to sort the file first for it to check for duplicates, and only return the unique lines via *-u*)

**Level 9:**  password in data.txt is human-readable string and preceded by equal signs.

1. *strings data.txt | grep “===”* (*strings* filters just the human-readable text, *grep* for the symbols)

**Level 10:** base64 encoded data in file

1. *base64 -d data.txt* (easy)

**Level 11:** data in file is Rot13

1. *cat data.txt | tr -A-Za-z N-ZA-Mn-za-m* (use *tr* to rotate the letters by 13 characters).

**Level 12:** a hexdump of a file has been repeatedly compressed.

1. Make a temporary directory: *mktemp -d*
2. Assign a variable name “temp” to the temp directory.
3. Revert the hexdump to original form: *xxd -r data2.bin data2*
4. *file data2* returns a Gzip format.
5. *mv data2 data2.gz*
6. *gzip -d data2.gz* 
7. *file data2* returns a Bzip format.
8. *bzip2 -d data2*
9. *file data2.out* returns Gzip format again.
10. *mv data2 data2.gz*
11. *gzip -d data2.gz*
12. *file data2* returns a Tar archive.
13. *tar -xf data2* results in data5.bin
14. *file data5.bin* returns another Tar achive.
15. *tar -xf data5.bin* results in data6.bin.
16. *file data6.bin* returns another Bzip format.
17. *bzip -d data6.bin*
18. *file data6.bin.out* returns another Tar archive.
19. *tar -xf data6.bin.out* results in data8.bin
20. *file data8.bin* returns another Gzip format.
21. *mv data8.bin data9.gz*
22. *gzip -d data9.gz*
23. *file data9* FINALLY returns ASCII text, which contains the password.

**Level 13:** use the SSH key to log in as bandit14.

1. Copied the key into a file *key*.
2. *chmod 600 key*
3. *ssh -i key bandit14@$bandit -p 2220*

**Level 14:** submit bandit14’s password to port 30000

1. *find / -type f -name bandit14 2>/dev/null →* returned */etc/bandit_pass/bandit14*
2. *echo <password> | nc [localhost](http://localhost) 30000*
3. It replied with the next password.

**Level 15:** submit the current password to port 30001 using SSL/TLS encryption.

1. *cat /etc/bandit_pass/bandit15*
2. Used *openssl* to connect to the port in an encrypted session: *openssl s_client -connect 127.0.0.1:30001*
3. Submitted the password and it returned the next one.

**Level 16:** find a listening port that speaks SSL/TLS and submit the password to it.

1. Made a temporary directory to write out my scripts: *mktemp -d*
2. Set a variable to the temp folder and cd’ed into it.
3. *nano [script.py](http://script.py) && chmod 700 script.py*
4. Python Script to see which ports are open in the range 31000-32001:
*import socket

ip = “127.0.0.1”
port_range = range(31000,32001)

openPorts = []

for port in port_range:
    sock = socket.socket(socket.AF_INET,socket.SOCK_STREAM) # create a socket
    result = sock.connect_ex((ip,port)) # attempt to connect

    # If the connection is successful, append to the list:
    if result == 0:
        openPorts.append(f”Port {port} is open”)
    sock.close # close the connection

print(”\n”.join(openPorts))*
5. There were only 5 ports open, so I just tested each of them manually to see if they successfully connected with *openssl:*
    1. Open Ports: 31046, 31518, 31691, 31790, 31960
    2. *openssl s_client -connect 127.0.0.1:<port>*
    3. Only 31518 and 31790 accept the input, but there’s a KEYUPDATE generated.
6. Bypassed the KEYUPDATE error with the *-ign_eof* tag and submitted the password successfully to port 31790: *echo <password> | openssl s_client -connect 127.0.0.1:31790*
7. It returned the private key.

**Level 17:** password is the only line that’s different between the two files.

1. *diff [passwords.new](http://passwords.new) passwords.old*

**Level18:** password is in a readme file, but I’m logged out the moment I SSH in.

1. I just passed in a cat command to the SSH connection: *echo “cat readme” | ssh bandit18@$bandit -p 2220* → it returned a line of text, which should be the password.

**Level19:** use the binary file to get the next password.

1. *strings* returns a few interesting things:
    1. a line of text “Run a command as another user.”
    2. bandit20.c → I’m wondering if the binary references this file.
    3. There’s also a location given /usr/bin/env, so maybe this is where the C file is?
2. Wow, I made this way too complicated…it was as simple as doing what the program said and running the cat command through the binary: *./bandit20-do cat /etc/bandit_pass/bandit20*

**Level20:** use the binary file to retrieve the password.

1. What *suconnect* file does: connects to [localhost](http://localhost) on a specified port → reads a line of text from the connection → compares the line to bandit20’s password → if it’s a match, it transmits bandit21’s password.
2. The question is what port to give as the argument, so I copied over my port scanner script again and ran it → there are 21 ports open, not including SSH (22).
3. I tried an imperfect Python script that sends the password over to each open port and reads what the port sends back, but this didn’t work.
4. I figured out I was going about this all wrong and the port number doesn’t matter…the binary just sends over the data to the given port. I had to peek at the solution for this one sadly, as I didn’t know how to do this, and it ended up being fairly simple:
    1. Pass the password data into a created netcat listener and background the whole process: *cat /etc/bandit_pass/bandit20 | nc -lvnp 1234 &*
    2. Check the process is there in *ps aux*.
    3. Connect to the running listener port via the binary: *./suconnect 1234*
5. The program successfully connected to the port, compared the password that was passed into the netcat session, and returned the next password.

**Level21:** crontab is running a program

1. *cat /etc/cron.d/cronjob_bandit22* runs the script /usr/bin/cronjob_bandit22.sh
2. The bash script changes permissions on a temp file, then cats out bandit22’s password to that file.
3. I read the file referenced in that script and got the password → as simple as that.

**Level22:** another cronjob challenge

1. *cat /etc/cron.d/cronjob_bandit23* → every reboot, it runs /usr/bin/cronjob_bandit23.sh
2. *cat /usr/bin/cronjob_bandit23.sh* → takes in a username and copies the password file for that username over to a temp directory
3. The script isn’t writeable, but it can be run by bandit22 and I believe the key is to trick it into thinking we’re bandit23, as it runs the “whoami” command to get the username.
4. I tried making variables for both whoami and myname, but this didn’t work.
5. Then it struck me that I could possibly make my own copy of the script with the same title as the existing one, change the PATH to start with the location of my own script, and then when the job runs next, it will run my script first (with root privileges of course):
    1. *mktemp -d* and went into that temp directory.
    2. Copied over the script code into my own file with the same name as the original.
    3. *export PATH=/tmp/<my directory>:$PATH* → prepended the PATH variable with my directory so that this would be the first folder it would look for the script.
6. So I just changed the line that specifies the myname variable to the value “bandit23”, ran the script, and it copied over the password to a temporary file that I was then able to read.
7. I’m extremely proud of myself right now 😊

**Level23:** time-based job is running a command.

1. *cat /etc/cron.d/cronjob_bandit24* → script that gets run in “/usr/bin/cronjob_bandit24.sh
2. The script shows that everything within the directory /var/spool/bandit24/foo gets run periodically.
3. Made a simple reading script:
    
    *#!/bin/bash*
    
    cat /etc/bandit_pass/bandit24 > /tmp/<my directory>/pass
    
4. Added executable privileges on my script: *chmod 777 script.sh*
5. Made a file called “pass” in the temp directory and changed its permissions: *chmod 666 pass*
6. Copied over the script to /var/spool/bandit24/foo: *cp [script.sh](http://script.sh) /var/spool/bandit24/foo*
7. Waited for it to run….And it didn’t.
8. After a lot of digging, I found out it was because I needed to set the permissions of my temp directory, too: *chmod 777 <my directory>*. Ran it again and it worked.

**Level24:** bruteforce the 4-digit passcode for the port to get the password.

1. Wrote a simple bash script to send a range of 4-digit numbers and bandit24’s password to the netcat port and send the output from the server to a text file:
    
    *#!/bin/bash
    
    for num in {1000..9999}
    do
        echo <password> $num
    done | nc [localhost](http://localhost) 30002 > results.txt*
    
2. *tail results.txt* revealed the password at the end.

**Level25:** log into bandit26 with the key in the current directory; connects to the next level.

1. *ssh -i key bandit26@$bandit -p 2220* → tried this but it crashes
2. *cat /etc/passwd* to see what kind of shell bandit26 has → */usr/bin/showtext*
3. *cat /usr/bin/showtext:*
    
    export TERM=linux
    exec more ~/text.txt
    exit 0
    
4. This level was way above me, so I had to refer to an online source.
5. The *more* command is used here, which is basically the entry point to gain a regular shell, and to prompt *more* to display, the terminal needs to be shrunk to a very small size.
6. Once *more* is triggered, V can be pressed to launch a text editor.
7. *:e* enters edited mode, which can be used to read the password → *:e /etc/bandit_pass/bandit26*

**Level26:** break out of the shell to get bandit27’s password.

1. Continuing from the previous level, the other option would be to convert to a regular bash shell.
2. *:set shell=/bin/bash* will change the shell to Bash, and *:shell* will open the current shell, so we can use *cat* like normal.
3. There is a simple binary in the current directory where you can run a command as bandit27, so I used that to grab the password: *./bandit27-do cat /etc/bandit_pass/bandit27*

**Level27:** git repository contains the password.

1. Created a temp directory and cloned the specified repository there: *git clone ssh://bandit27-git@localhost:2220/home/bandit27-git/repo*
2. *cd repo* → *cat README* (easy)

**Level28:** git

1. Same first step as previous level, but for bandit28’s repo.
2. *cd repo* → *cat [README.md](http://README.md)* → it contains notes for the next level, including a placeholder for the password.
3. My first thought is that the password was written here before the current version of the file, so I’ll check the history → *git log -p — README.md*
4. And the previous version did indeed have the password.

**Level29:** git

1. Same first step as previous 2 levels, but for bandit29’s repo.
2. *cd repo* → *cat [README.md](http://README.md)* → it says no passwords in production, but I’ll double-check the history. Yes, it returns nothing.
3. *git log —al* → checked history of commits for all of the branches.
4. Found a commit with the note “add data needed for development”.
5. *git show <commit>* returned the original data for the password line, which is the actual password.

**Level30:** git

1. Same first step as previous 3 levels, but for bandit30’s repo.
2. There is nothing in the logs this time, and there’s only one branch.
3. After trying various commands, *git tag* returned a tag called *secret*.
4. *git show secret* returned what looks to be the password.

**Level31:** git

1. Same first step as previous 4 levels, but for bandit31’s repo.
2. The README file this time says that the task is to push a file called “key.txt” with the content “May I come in?” to the remote repository’s master branch.
3. Created the file key.txt with the appropriate content.
4. *git add key.txt -f* to force the file to be put into pending commits.
5. *git commit -m “May I come in?”* (since that’s the actual content of the file.
6. *git push -u origin master* → the message displayed as confirmation of the commit’s receipt was the password for the next level.

**Level32:** the only thing is says is this is an “escape.”

1. It says we are using a n “UPPERCASE SHELL”
2. Many commands are restricted, and it’s converting the commands to uppercase form.
3. I tried to see if environment variables could be assigned, but this didn’t work.
4. The hint given is the command ‘sh,’ which seems to be related to a Bash interpreter.
5. After reading the man page, I’m sure the key is in variables.
6. It ended up being as simple as switching to a regular shell via $0.

And that’s it! Did about 95% on my own this time :)

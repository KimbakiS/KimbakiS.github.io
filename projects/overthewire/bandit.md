### Informal writeup, collection of notes as I made my way through the Bandit challenge from OverTheWire, which I revisited after investing some more time in learning scripting.

**Level 1:** 

SSH into the game as bandit 0 on port 2220

![image.png](attachment:d1e00407-668c-4523-b98d-cf08cedd37d0:image.png)

Cat the ***readme*** file

![image.png](attachment:a9d12c4b-02f2-494b-bb1d-c17451c2e128:5369a535-a0a1-492e-93ff-83fe89ff2f07.png)

**Level 2:**

SSH as bandit1 and cat the   **`*-*`**   file

![image.png](attachment:6a8867a5-6d6f-4f3a-b691-e684a3894a76:image.png)

What I learned:

- I used **`find`** to see how to write this out → in short, you apparently prefix with `*./*` to cat out files beginning with symbols.

**Level 3**: 

SSH as bandit2 and cat the filename with the spaces:

![image.png](attachment:1fda3249-0fb9-4b43-b4bd-a09ae3eab4e7:image.png)

**Level 4:**

SSH as bandit3, CD into directory, and cat the hidden file

![image.png](attachment:a78defc6-9571-4af2-8ccb-501937ca9fee:image.png)

**Level 5:**

SSH as bandit4 and retrieve the only human-readable text from the list of files:

![image.png](attachment:12bb933b-46ee-4295-a5da-9285df23db5c:image.png)

Since the files were each very small, I simply catted all of them out to save time **(not pretty, but it works).

**Level 6:** 

SSH as bandit 5 and find a file that’s 1033 bytes, is not executable, and is human-readable.

![image.png](attachment:7f4cb103-51b9-45cf-ad6d-41683ef175a2:image.png)

Managed to find the right file matching only two of the conditions:

- `-size 1033c` limits the results to files with 1033 bytes
- `\! -executable` shows only files that are NOT executable, with the `\!` acting as a negation.

**Level 7:**

   Find a file that is owned by bandit7, is in group bandit6, and is 33 bytes

![image.png](attachment:34c0412e-427b-4e19-99b9-05b993dbdfb2:image.png)

- `2>/dev/null` avoids the “Permission Denied” errors by dumping such output into garbage space

**Level 8:**

Find the password next to the word “millionth” in data.txt

![image.png](attachment:865092ff-2cd8-470a-b7e0-6fbdfd50c832:image.png)

**Level 9:**

The password is the only line that occurs just once in data.txt

![image.png](attachment:1ec271d2-9533-447e-a19b-f1fd9a3b694f:image.png)

You naturally have to `sort` the file first for it to check for duplicates, and then, only return the unique lines via `uniq -u`)

**Level 10:** 

Password in data.txt is human-readable string and preceded by equal signs.

![image.png](attachment:a489355e-f525-4b69-b90e-0e8798e963b8:image.png)

- `strings` filters for just the human-readable text
- `grep` can filter for a specific pattern, with the wildcard (`*`) representing ‘anything’

**Level 11:**

Decode the base64-encoded data

![image.png](attachment:33324833-373d-4c93-9177-a766b4ea53d6:image.png)

**Level 12:**

The data in the file is Rot13

![image.png](attachment:6d16c29e-9f90-4e97-b78d-2f1d5e9461e2:image.png)

`tr` can be used to ‘translate’ (or ‘rotate’) the letters by 13 characters.

- Logic: you essentially swap the characters, starting with the letter ‘A’ (the 1st letter in the alphabet) and the letter ‘N’ (the 14th letter in the alphabet, or 13 letters ahead of ‘A’).
- Both lower and uppercase letters have to be swapped separately.

**Level 13:  a** hexdump of a file has been repeatedly compressed.

Make a temporary directory to work with the file

![image.png](attachment:104cff4a-5612-48e2-9170-beeab00b2bdc:image.png)

Revert the hexdump to its original form:

![image.png](attachment:c202bd82-2491-46f9-8a47-a9cfa4182823:image.png)

Before it was converted to a hexdump, the file a Gzip called “data2.bin”

It took many, many iterations to decompress the file, and it involved the following process:

1. Display what type of file it is with `file`
2. If in Gzip format: `mv data1 data2.gz` → `gzip -d data2.gz`
3. If in Bzip format: `bzip2 -d data2`
4. If in Tar format: `tar -xf data2`
    
    ![image.png](attachment:7088ff5d-68b5-4d6a-8fb0-594beac0123c:726b86dd-abcf-48cd-b9fa-57a470e074ee.png)
    
    At data9, we FINALLY have ASCII text, which contains the password.
    

![image.png](attachment:d3422ff0-7b5d-4fdf-b067-6a3eb3fbb209:image.png)

**Level 14:** use the SSH key to log in as bandit14.

This is a classic RSA key

![image.png](attachment:2bcf2733-82eb-4610-bd22-9cd2d1c48a01:image.png)

Copied the key into a file **bandit14.rsa,** modified the permissions to be able to use it, then SSHed as bandit14 with the key:

![image.png](attachment:04869dcb-609c-4e8d-b014-57780944f244:image.png)

**Level 15:** submit bandit14’s password to port 30000

The previous level mentioned where the password was, so we just connect to the port via Netcat and send over the contents of the password file through a pipe (`|`) and get its reply:

![image.png](attachment:8eb5277e-e8e9-43f4-a4c8-7d780b559c33:image.png)

**Level 16:** submit the current level’s password to port 30001 using SSL/TLS encryption.

Have to use `openssl` to connect securely (with an encrypted session) to that port:

![image.png](attachment:acd37fc6-2d1f-4c88-8203-ad3245e08c9b:image.png)

Simply submitted the bandit15’s password and it returned the next one:

![image.png](attachment:3ef58870-7404-4fe1-9aec-cb0426d94846:image.png)

**Level 17:** find a listening port that speaks SSL/TLS and submit the password to it.

Easy method would be to run Nmap locally to find open ports in the range given:

![image.png](attachment:9f325e9d-a508-411d-89e6-f91476ee9261:image.png)

The more challenging method is to write out your own script, which is useful for when you don’t have port scanning tools such as Nmap available on the local system.

So, I made a temp directory to write out the script:

![image.png](attachment:ee637e14-8b5f-445a-b51d-fd5d45dae34a:image.png)

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

![image.png](attachment:c435703f-86bd-407c-9ed8-ce242000c875:image.png)

Open Ports: 31046, 31518, 31691, 31790, 31960

Since there are only 5 ports open, I just tested each of them manually to see if they successfully connected with openssl:

![image.png](attachment:f498cd3a-02fc-4eeb-9df0-d17e1f5551cd:image.png)

Both 31518 and 31790 accept input, but there’s a KEYUPDATE generated, so I had to do some research into this error and ended up finding a parameter that can bypass it `-ign_eof`:

![image.png](attachment:51fd043f-9bc7-401d-82bb-11274d3a7ea8:image.png)

Port 31790 accepted the password and returned a private key

![image.png](attachment:bf9d6b68-1d85-47e3-b304-6ffc8c0899ec:image.png)

**Level 18: the** password is the only line that’s different between the two files.

Simply ran a `diff` against the two password files to find the change:

![image.png](attachment:77dd09e3-eef2-4fc0-8567-a639ad769b8d:image.png)

**Level 19:** password is in a readme file, but it logged me out the moment I logged in.

We should still be able to pass commands to the system through SSH even if we can’t use an interactive terminal, so since we know the name of the file we want, can just cat that:

![image.png](attachment:b197eb49-0df0-428d-9f41-9e0880542c60:image.png)

**Level 20:** use the binary file to get the next password.

1. `strings` returns a few interesting things:
    1. Line of text: “`Run a command as another user`.”
    2. `bandit20.c` → I’m wondering if the binary references this file.
    3. There’s also a location given `/usr/bin/env`, so maybe this is where the C file is?

![image.png](attachment:a2f7fbe9-882d-479e-b613-58ed2378e3b3:image.png)

Since the file’s name references the bandit20 user, maybe all we need to do is run a cat command through the binary:

![image.png](attachment:6b68b52d-26f4-4e6f-b120-8752b4b1e4dd:image.png)

Yep…it was that simple. Note to self to always run the program BEFORE trying to reverse engineer it.

**Level 21:** use the binary file to retrieve the password.

According to the level notes, the binary `suconnect` does the following: connects to [localhost](http://localhost) on a specified port → reads a line of text from the connection → compares the line to bandit20’s password → if it’s a match, it transmits bandit21’s password.

Since the binary needs to retrieve the text for the comparison from a port connection, we probably need a way to store text in a port in the first place. I was not familiar with how to do this, so I had to do some research and discovered that we can actually do just that with Netcat:

![image.png](attachment:b4405a12-acae-4c3a-be64-619301f78974:image.png)

Essentially, we create a listening port 4444 and pass bandit20’s password to it, background the listener, and then have the binary access that listening port to retrieve the text.

And because the lines match, we do indeed get bandit21’s password.

**Level 22:** crontab is running a program

Found a script being run at reboot: `/usr/bin/cronjob_bandit22.sh`, which is a bash script that changes permissions on a temp file, then cats out bandit22’s password to that file.

![image.png](attachment:ab1fce63-b6f5-4582-a3be-4c864c4f00b9:image.png)

Simply read the file referenced in that script and got the password.

**Level 23:** another cronjob challenge

Similar process to before, located a binary that copies the password files for a username provided by the `whoami` prompt over to a temp directory.

![image.png](attachment:a50ba3cc-1e5c-491c-9fd7-c38be68cda2c:image.png)

The script isn’t writeable, but it can be run by bandit22 and I believe the key is to trick it into thinking we’re bandit23, as it runs the `whoami` command to get the username.

I tried making variables for both whoami and myname, but this didn’t work.

Then it struck me that I could possibly make my own copy of the script with the same title as the existing one, change the PATH to start with the location of my own script, and then when the job runs next, it will run my script first (with root privileges of course).

1. So, I created a temp directory and prepended the PATH variable with that directory.

![image.png](attachment:6fbf295f-54bc-4929-a30e-45b6b2b88573:image.png)

1. I copied over the script code to my own file with the same name as the original, changing the line that specifies the **myname** variable to the value “bandit23” instead of performing `whoami`:
2. And now, the temp directory is the first place that will be checked for the script.
3. Finally, the script details that the name of the file that will contain bandit23’s password is an MD5 hash, so I ran that line of code separately to get the exact value, then catted out the file:
    
    ![image.png](attachment:a22351c1-a348-4e20-ac3f-ba0bcee591e2:image.png)
    
4. I’m extremely proud of myself right now 😊

**Level 24:** time-based job is running a command.

1. The script shows that everything within the directory /var/spool/bandit24/foo gets run periodically.
2. Made a simple reading script:
    
    ![image.png](attachment:ffdf3bb5-bb72-4936-9629-ce673fc2c11b:image.png)
    
3. Added executable privileges on my script.
4. Made a file called “bandit25” in the temp directory and changed its permissions, too.
5. Copied over the script to /var/spool/bandit24/foo and waited for it to run…And it didn’t
6. After a lot of digging, I found out it was because I needed to set the permissions of my temp directory, too: *chmod 777 <my directory>*. Ran it again and it worked.
    
    ![image.png](attachment:2547e0f1-60cd-44be-867b-47cdf847d075:image.png)
    

**Level 25:** bruteforce the 4-digit passcode for the port to get the password.

Wrote a simple bash script to send a range of 4-digit numbers and bandit24’s password to the Netcat port and send the output from the server to a text file:

```bash
#!/bin/bash

for num in {1000..9999}
do
    echo <password> $num
done | nc [localhost](http://localhost) 30002 > results.txt
```

Add executable permissions to the script and run it.

![image.png](attachment:d8038321-e6d4-4c21-aad4-db89c64a7ead:image.png)

Grep out the wrong answers to highlight the correct  one

![image.png](attachment:b3946d94-d824-46c4-bd85-48d0047b6733:image.png)

**Level 26:** log into bandit26 with the key in the current directory; connects to the next level.

1. *ssh -i key bandit26@$bandit -p 2220* → tried this but it crashes
2. Checked to see what shell bandit26 is using, which is `showtext`, and looking at the binary for this interesting shell, it looks like it displays a text file with the more command, then immediately exits

![image.png](attachment:497807c1-ddad-4b62-b6ff-d4f0ff41c2e1:image.png)

1. I did realize that the `more` command would play a role here, but it took a lot of digging to realize that it’s basically the entry point to gain a regular shell. And to prompt it **to display, the terminal needs to be shrunk to a very small size:

![image.png](attachment:05c88164-a26b-4bf3-8d94-288fe9d56ee9:image.png)

1. Once this is triggered, `V` can be pressed to actually open up Vim and return to normal windows size, after which we can change the shell being used by bandit26 to simple Bash via `:set shell=/bin/bash`, switch to it, and finally cat out bandit26’s password:
    
    ![image.png](attachment:f0bcf099-abde-4466-888b-9777fd45a6c1:image.png)
    

**Level 27:**

There is a simple binary in the current directory where you can run a command as bandit27, so I used that to grab the password (most of the work was done in the previous level):

![image.png](attachment:74157dd3-c12b-4ed3-b129-305246a80d64:image.png)

**Level 28:** git repository contains the password.

1. Created a temp directory and cloned the specified repository there, then just catted the README file:
    
    ![image.png](attachment:98e2f112-fb75-4a7a-982b-3842b7194036:image.png)
    

**Level 29:** git

1. Same first step as previous level, but for bandit28’s repo.
2. The [README.md](http://README.md) file contains notes for the next level, including a placeholder for the password.
3. My first thought is that the password was written here before the current version of the file (because of the placeholder), so I’ll check the history:
    
    ![image.png](attachment:ec00fe5a-60e1-45de-b80a-bcaef7bdbe13:image.png)
    
4. And the previous version did indeed have the password.

**Level 30:** git

1. Same first step as previous 2 levels, but for bandit29’s repo.
2. *cd repo* → *cat [README.md](http://README.md)* → it says no passwords in production, but I’ll double-check the history. Yes, it returns nothing.
3. *git log --all* → checked history of commits for all of the branches.
4. Found a commit with the note “add data needed for development”.
5. *git show <commit>* returned the original data for the password line, which is the actual password.

![image.png](attachment:61761731-a6c1-4580-a8aa-d8d3eca044d1:image.png)

**Level 31:** git

1. Same first step as previous 3 levels, but for bandit30’s repo.
2. There is nothing in the logs this time, and there’s only one branch.
3. After trying various commands, *git tag* returned a tag called *secret*.
4. *git show secret* returned what looks to be the password.
    
    ![image.png](attachment:2f3ec6ea-95dd-489b-9234-30912d96622e:image.png)
    

**Level 32:** git

1. Same first step as previous 4 levels, but for bandit31’s repo.
2. The README file this time says that the task is to push a file called “key.txt” with the content “May I come in?” to the remote repository’s master branch.
    
    ![image.png](attachment:f4f93483-9b2d-4916-aa45-c609043fa944:image.png)
    
3. Pushed the file to the remote repository, and the message displayed as confirmation of the commit’s receipt was the password for the next level.

![image.png](attachment:03327a14-3b16-4a2b-83a2-515142573dbc:image.png)

![image.png](attachment:7058cdf9-1417-47f0-8f16-64e1f96bf593:image.png)

**Level 33:** the only thing it says is this is an “escape.”

1. It says we are using a n “UPPERCASE SHELL”
2. Many commands are restricted, and it’s converting the commands to uppercase form.
3. I tried to see if environment variables could be assigned, but this didn’t work.
4. The hint given is the command ‘sh,’ which seems to be related to a Bash interpreter.
5. After reading the man page, I’m sure the key is in variables.
6. It ended up being as simple as switching to a regular shell via $0.

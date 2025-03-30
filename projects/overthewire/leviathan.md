# OverTheWire's Leviathan Challenge

### Collection of screenshots and notes

Password directory: /etc/leviathan_pass/
Server: [leviathan.labs.overthewire.org](http://leviathan.labs.overthewire.org): 2223

Challenge notes and hints will all be in the home directories of the current level.

**Level 0:**

SSHed as leviathan0 and found a folder containing a simple HTML file of a bookmarks export.
![image](https://github.com/user-attachments/assets/68d57112-5437-4c14-a292-38c7f0e75fc4)

Could have tried other keywords to locate the password, but one of the most obvious ones to me was “leviathan”, since the password is likely stored in the leviathan server:
![image](https://github.com/user-attachments/assets/ea51cd27-8347-46a8-b728-8393d2665c53)

**Level 1:** 

Found a file with SUID (`s`) privileges and a prompt for a password.
![image](https://github.com/user-attachments/assets/f9abc6e2-8877-4b14-b6fb-7a7f37557352)

It appears to be an executable program written in C where you input the correct password to get a shell; the strings are compared using the strcmp function, and there are a couple of keywords right before you input the password:
![image](https://github.com/user-attachments/assets/70e867f2-26c9-4b1a-b399-6163063479a9)

Grepped for the word “love” in the program, which originally didn’t return anything because of binary matches, so I allowed the binary matches via -a parameter and got 4 potential keyword matches:
![image](https://github.com/user-attachments/assets/f7cc9517-611e-4015-a1dc-8a1fb672f0c7)

And one of the keywords did end up being the correct one, so we got a shell and catted the password:
![image](https://github.com/user-attachments/assets/97f04acc-572a-49b2-8fe2-9072931ac99d)


**Level 2:** 

There’s a program with SUID privileges that appears to print out a file you specify, so just in case, I tried to retrieve the password through it, but it unsurprisingly requires you to have the right permissions for access:

![image](https://github.com/user-attachments/assets/cd7361b6-bc7e-4cc1-90b0-6fd933fad276)

What’s interesting is that even if the current user owns the file, we get a “Permission denied” message.” Only files within the current directory are successfully printed out.

Through online research, I discovered a program called `ltrace` that can trace library calls made within a program, but it didn’t return anything interesting, so I just eyeballed the program’s strings and located a keyword of interest, “access”, since I didn’t recognize it as a command or typical C function.

![image](https://github.com/user-attachments/assets/faa7506d-2cfd-41c5-969f-67c83ef99024)

I looked in the man pages in case it was a command of some sort and discovered that it is actually a C library that checks whether the calling process can access a given file:
![image](https://github.com/user-attachments/assets/0940618f-6692-4a26-a99b-b07328bdfc8d)

This suggests that there is likely not input sanitization, so I tried to create an empty with a filename that attaches the bash command to the end of it, then referenced that file as the argument to the printfile program, and it ran exactly as input, opening a bash shell as leviathan3:
![image](https://github.com/user-attachments/assets/93f93cbb-4955-4f4d-a791-98c9a2cde2f0)


**Level 3:** 

The file “level3” seems to open a shell if you input the right password.

![image](https://github.com/user-attachments/assets/232750e0-652b-4954-88ea-589337c5ca55)

Running an ltrace again on the program (starting to LOVE this new command!), we see that a strcmp is run, comparing the input with a plaintext value “snlprintf.”
![image](https://github.com/user-attachments/assets/0f418b8c-49a9-45dd-94dc-3d2bb23d93d4)

So, we run the program again and enter this as the password and get a shell as leviathan4.

*Note to self: why did you run the program with a filename? You’re still stuck on the previous level -_-*


**Level 4:**

There is a simple file in the “.trash” directory that contains binary.
![image](https://github.com/user-attachments/assets/c8665287-0feb-4a06-a251-73c0bcf3db41)

We just crack the binary with CyberChef and it looks like the password:
![image](https://github.com/user-attachments/assets/b3a970e9-0a6f-4d21-a322-a80c01bff9bc)

 But it can’t be that easy, right?
 
![image](https://github.com/user-attachments/assets/cf3da40f-f210-4da4-9c3f-8ba631cfb75a)

Yep…..it is certainly that easy.

**Level 5:**

There’s a program “leviathan5” with the owner set to “leviathan6”, and it looks like it just opens a particullar file “/tmp/file.log”, but the file doesn’t currently exist:
![image](https://github.com/user-attachments/assets/2a0b9094-fcce-445e-9442-ee0950d1828e)

I created a file at that exact path and it looks like the program does indeed read what’s in there.
![image](https://github.com/user-attachments/assets/ffc0f24d-b2c2-4a6c-8f3a-fe29bde08fdd)

We can actually create a Symlink (with the ln -s command) between “file.log” and the password file, run the program, and it (hopefully) will print out the password file:
![image](https://github.com/user-attachments/assets/b0069fe2-7806-4efe-ac09-ac811d8d83ce)

Logic: basically, file.log is a link to the password file, so when fopen tries to retrieve the file.log, it in fact prints out the file it links to.


**Level 6:**

Pretty straightforward program that asks for a 4-digit code:
![image](https://github.com/user-attachments/assets/948701b3-2ecc-46dd-8c37-591f1ba9235b)

This is probably a matter of using a simple bruteforce script that I wrote in Bash (not the prettiest, but it was functional, except for not exiting after the else condition was met…)

```bash
#!/bin/bash

cd ~

for num in {1000.9999}
do
	if ./leviathan6 $num | grep -q "Wrong"; then
		echo $num
		continue
	else
		echo "Correct: " $num
		exit 1
fi
done
```

And it stopped right before the correct code, which we plug into the program and get a shell.

![image](https://github.com/user-attachments/assets/2d2b7fa9-99dc-4e24-8b55-77965db754a6)


And that's it! Kind of wish this one went on a bit longer...


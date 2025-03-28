**Level0:** ssh as leviathan0 and find next password.

1. *ls -la*
2. *cd .backup*
3. *cat bookmarks.html | grep pass*
4. “The password for leviathan1 is …”

**Level1:** 

1. *file check* → C executable program where you input the correct password to get a shell.
2. *string check* → returned a couple of notable words, one partial (secr, love).
3. *cat check | grep love* didn’t work because of binary matches.
4. A simple *cat* of the program and visual scan found 4 possible words, of which the first was the password to get leviathan2’s shell.
5. *cat /etc/leviathan_pass/leviathan2*

**Level2:** 

1. There’s a program, _**printfile**_, that does exactly that: prints out a file you specify (it essentially calls */bin/cat*), as long as you have the permission to do so.
2. What’s interesting is that even if the current user owns the file, we get a “Permission denied” message.” Only files within the current directory are successfully printed out.
3. Found out online about a command called *ltrace* that can “trace” the library calls used in the program, and from here I found out a function called *access* is used on the file we’re trying to print → *access(”/etc/leviathan_pss/leviathan3”, 4)*
4. *man access* → it’s a C library that checks whether the calling process can access a given file.
5. There apparently is not input sanitization, so I was able to make a filename that attaches a command at the end of it: *<temp directory>/test.txt;bash*
6. *printfile /<temp directory/test.txt;bash* ran it exactly as input and so opened a bash shell as leviathan3.
7. *cat /etc/leviathan_pass/leviathan3*

**Level3:** 

1. The file “level3” seems to open a shell if you input the right password.
2. Running an *ltrace* (LOVE this new command!) on the program, we see that a strcmp is run, comparing your input with a plaintext value “snlprintf,” which is actually the right password.
3. *./level3* → *snlprintf* (input) → We got a shell and cat out the next password.

**Level4:**

1. *cd .trash* → *cat bin* → looks like this program contains leviathan5’s password.
2. Running the program, the output is binary, so a simple CyberChef binary cracking returned what looks like our password….it can’t be that easy, right?
3. *Tests the password out. Yes, it is indeed that easy. Wow…this should have been level 1.

**Level5:**

1. The file *leviathan5* looks like it just opens a particular file /tmp/file.log, but the file doesn’t currently exist.
2. I created a file at that exact path and it looks like the program does indeed read what’s in there.
3. We can actually create a symlink between file.log and the password file, run the program, and at will print out the password file.
4. *ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log* → basically, file.log is a link to the password file, so when *fopen* is called on file.log, it in fact prints out the file it links to.

**Level6:**

1. Pretty straightforward program that opens a shell if you guess the correct 4-digit code.
2. This is probably a matter of using a simple bruteforce script that I wrote in Bash (not the prettiest, but it was functional, except for exiting after the else condition was met…)
    
    #!/bin/bash
    
    *cd /home/leviathan6*
    
    *for num in {1000.9999}
    do*
    
    *if ./leviathan6 $num | grep -q “Wrong”; then*
    
    *echo $num
    echo “Wrong: “ $num >> <temp directory>/output.txt
    continue*
    
    *else*
    
    *echo “Correct: “ $num >> <temp directory>/output.txt
    exit 1*
    
    *fi*
    
    *done*
    
3. Catting the output file, we see the correct password for the program is 7123, so we give that as the input and get a shell as leviathan7.

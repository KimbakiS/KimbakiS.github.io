---
layout: default
---


**Scenario:**

A ticket was opened pertaining to a potential malware threat. The ticket contains multiple file attachments that are presumed to be malware samples.

### PREPARATION

Downloaded the malware samples from intelligence site and extracted them in their own directory:

![image](https://github.com/user-attachments/assets/9771ebd6-d7d0-405d-9cb8-5c49fed1c14a)

There are 5 DLL samples in total

Created a simple Bash script to retrieve the SHA256 hashes of the DLLs (there are only 5 files, but this should be standard practice anyway):

![image](https://github.com/user-attachments/assets/1b415620-3a52-4c48-b803-504378fdd649)

I also modified the script to get the SHA1 hashes in another file, but that is redundant to picture here.

### SAMPLE: `pRsm.dll`

VirusTotal mass flags this DLL as a Trojan

![image](https://github.com/user-attachments/assets/6ab83893-ab85-4d37-9309-a233f45b8812)

Opening the sample in VirusTotal’s graph editor, we see that the DLL itself is a bundle of 5 files and, in one of the attack instances, there was a name given that looked like it may be the name of the malware framework, so I looked it up and confirmed this: MgBot

![image](https://github.com/user-attachments/assets/92da63c7-0757-46da-bde6-9a84fbd7a6f7)

### APT GROUP: `Daggeryfly` (aka, `Evasive Panda`)

And the APT group appears to be called both Daggerfly and Evasive Panda, and this APT is known for using the MgBot malware framework to spy on victims for cyberespionage purposes:

![image](https://github.com/user-attachments/assets/efb29d09-51d0-4a00-9a60-4dd69f3ce0df)

*(fascinating article about the technical capabilities at this page:* [Evasive Panda leverages Monlam Festival to target Tibetans](https://www.welivesecurity.com/en/eset-research/evasive-panda-leverages-monlam-festival-target-tibetans/))

In line with the “spying” capabilities, according to the security article [Daggerfly: APT Actor Targets Telecoms Company in Africa | Symantec Enterprise Blogs](https://www.security.com/threat-intelligence/apt-attacks-telecoms-africa-mgbot), this particular DLL’s job is to capture audio from the system:

![image](https://github.com/user-attachments/assets/b19bc4b0-d53e-4b09-9286-57eaf8c65a38)

And MITRE ATT&CK returns the Technique associated with the audio capture in the MgBot malware framework analysis:

![image](https://github.com/user-attachments/assets/5b524553-dc9a-4e91-9db5-72c23b3cdf92)

According to the article at Evasive Panda APT group delivers malware via updates for popular Chinese software, the first malicious download link was:

![image](https://github.com/user-attachments/assets/325ef68e-6e0f-4000-bdf9-3ff7540af28b)

And this article also neatly displays what the functions of the 5 DLLs we have in our samples folder were:

- `cbmrpa.dll` — “captures text copied to the clipboard and logs information from the USBSTOR registry key.”
- `maillfpassword.dll` — “steals credentials from Outlook and Foxmail email client software.”
- `pRsm.dll` — “captures input and output audio streams.”
- `qmsdp.dll` — “a complex plugin designed to steal the content from the Tencent QQ database that stores the user’s message history. This is achieved by in-memory patching of the software component KernelUtils.dll and dropping a fake userenv.dll DLL.”
- `wcdbcrk.dll` — “information stealer for Tencent WeChat.”

In short, the malware steals info from the clipboard, audio, messages, email credentials, and apparently other assets that aren’t included in our particular scenario.

The APT group also naturally had set up a C2 botnet beforehand at the following IPs:

![image](https://github.com/user-attachments/assets/23203790-6d7f-4586-bf5c-a96bd407ef63)

Beyond Windows executables, it also looks like one of the C2 IP addresses has also been associated with Android files as well:

![image](https://github.com/user-attachments/assets/a0d5ae9f-5a77-4d54-92bb-7da35539f003)
![image](https://github.com/user-attachments/assets/efc88a25-6915-465e-9394-b3000b500790)

Badge for solving this challenge: [Friday Fixer Badge](https://tryhackme.com/KimbakiS/badges/friday-fixer).
![image](https://github.com/user-attachments/assets/d7f655ce-47b9-44f0-87cf-3b1064b9a92c)



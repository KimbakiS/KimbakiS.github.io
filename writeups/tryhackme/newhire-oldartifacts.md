---
layout: default
---

**Scenario:**

SOC analysis for any IoCs on endpoints using Splunk

**Skills:**

- Investigating intrusion attacks using Splunk.
- Analyzing endpoint events and identifying malicious activities in a managed security service provider (MSSP) environment

### Questions

1. Path of the binary “Web Browser Password Viewer”
2. Company Name
3. Name of suspicious binary running from same folder
4. The IP the binary made an outbound connection to
5. The path of the registry key the binary made a change to
6. Two binary names of processes that were killed
7. Last command executed when attacker ran commands in PowerShell to change the behavior of Windows Defender
8. The four IDs set by the attacker
9. Full path to the malicious binary executed from another AppData location
10. The DLLs loaded from that binary

### PREPARATION:

First, we have to narrow down our search to December 2021, which is when the security endpoint solutions were widely turned off. There are 27,378 events during this month across all indexes:

![image](https://github.com/user-attachments/assets/41b93449-ed68-4b5d-af5a-12f01488fb37)

*The ‘main’ index will be our focus for this room.*

### QUESTION 1 & 2: suspicious binary

Our starting point is a binary known as “Web Browser Password Viewer,” so let’s just assume that an alert went off for it and that we are beginning our investigation here.

![image](https://github.com/user-attachments/assets/e4516a7a-5ddd-43c8-8f6e-19157a48cc88)

The binary was an executable called `11111.exe` that loaded from an AppData path of the user `FINANC~1`, and both of these names are obviously strange enough to warrant further investigation.

### Question 3: suspicious binary from same location

Our next step is to look for other files that executed from the same location. There are 27 different matches, so I switched to a graph view to see the list of executables better:

![image](https://github.com/user-attachments/assets/37d1f1dc-074e-4d49-a268-1322e561b8b3)

Many of these matches seem to be maybe archives containing either DLLs or EXEs:

![image](https://github.com/user-attachments/assets/ecc21e4c-22d9-4da2-a5c2-002c998ce0da)

It turns out I was going about this the wrong way and was looking at the Image Loaded field, which is the physical location of the executable, instead of the Current Directory, which is the actual working directory at the time the process was launched:

On those lines, I had to switch my search:

![image](https://github.com/user-attachments/assets/3da166d7-8c3f-49f2-a3c3-9a79fc6e39e2)

And on those lines, we did find an executable that was launched from that directory:

![image](https://github.com/user-attachments/assets/7db3333e-2939-4603-865e-82d82d5d2d69)

![image](https://github.com/user-attachments/assets/fb4a0877-c16f-4aaa-a102-82d927dd68e7)

Here, the CommandLine field is the actual command used to execute the process, and the ParentCommandLine field would be the one used to execute the process responsible for *spawning* that process. So we should probably turn out attention to the parent process here.

There were 189 events associated with the process `IonicLarge.exe`, and I ended up looking at the earliest events to see how it was created. An executable `Setup.exe` was responsible for its creation, and the second earliest event showed the original file name:

![image](https://github.com/user-attachments/assets/3c321242-a8dc-4fbd-8e8d-89f172ad653e)

### **QUESTION 4: outbound communication**

Now, we have to focus our attention on the network connections that were associated with this executable `IonicLarge.exe`. Since we’re dealing with network events and Sysmon, I filtered the results by event code 3, which returned 87 results:

![image](https://github.com/user-attachments/assets/0db1496d-4744-4985-984b-4aeb9e464af6)

There were 8 destination IPs:

![image](https://github.com/user-attachments/assets/1dfd1c31-6c68-43ef-94f8-495d858d1fe7)

And the majority of the packets went to a port 8888, which is odd:

![image](https://github.com/user-attachments/assets/03872005-773d-4607-9281-038232ec6a4b)

I investigated the second IP in the list (naturally forwent the loopback address): `2.56.59.42`

### QUESTION 5: registry key changes

The Sysmon code for registry changes would be ‘13’, so I filtered the results accordingly, which returned 9 log entries:

![image](https://github.com/user-attachments/assets/de83e92a-5346-41f1-8eaf-073f432722cf)

There were several changes made within the Windows Defender key, including disabling anti-spyware and real-time protection, which is common practice for malware:

![image](https://github.com/user-attachments/assets/4b145437-f8f8-4447-9374-32fc8e52b3e0)

### QUESTION 6: killed processes

There were 2 processes that were killed via the CMD command “taskkill” (shared one here):

![image](https://github.com/user-attachments/assets/922a6c18-2dd3-46e8-8692-59905733d837)

### QUESTION 7 & 8: changes to Defender with PowerShell

Beyond the changes made in the associated registry key, the attacker apparently also used PowerShell to make 4 more configuration changes to Defender:

![image](https://github.com/user-attachments/assets/7f2465c4-9881-4929-8fd4-ecd7c9e52618)

The latest of these changes was reflected in the code: `powershell  WMIC /NAMESPACE:\\root\Microsoft\Windows\Defender PATH MSFT_MpPreference call Add ThreatIDDefaultAction_Ids=2147737394 ThreatIDDefaultAction_Actions=6 Force=True`

Upon outside research, I discovered that this command, in short, adds a rule to the Defender’s preferences that allows a specific threat with the given ID. In this case, this is a malicious action to enable the attacker to bypass yet another security layer.

The four threat IDs that were enabled were: `2147735503`, `2147737010`, `2147737007`, `2147737394`

### QUESTION 9 & 10: suspicious binary and its DLLs

Filtered for other binaries that executed from the AppData location and came across one particular binary that had the majority of events associated with it, indicating a lot of activity: `EasyCalc.exe`

![image](https://github.com/user-attachments/assets/52685f34-8566-4cf6-8c3e-3b109386e007)

And then filtered for the binary (and variations of it) and DLL files in particular and discovered 3 DLLs that were created by the `EasyCalc License Agreement.exe` file:

![image](https://github.com/user-attachments/assets/196ab7f8-c0c1-46ad-a9c4-878f425c4ce1)

![image](https://github.com/user-attachments/assets/6d0a6105-2857-4ffb-b768-c7e3afdfcc1c)

And with that, this room is completed! If I find time, I’ll try to go back and do some more investigation to put the pieces we’ve gathered here in a more cohesive timeline.

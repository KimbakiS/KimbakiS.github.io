---
layout: default
---

A simple room that involves investigations through each tier of the Pyramid of Pain.

### SCENARIO:

![image](https://github.com/user-attachments/assets/9b23400c-d344-42cb-b4c3-a31a50a69431)

This is a purple team scenario, and we’re acting on behalf of the blue team to test the security controls by seeing if they pick up the red team’s attack simulations.

### SAMPLE 1: HASHES

First sample scanned with Malware Sandbox:

![image](https://github.com/user-attachments/assets/b02020ea-8308-4e2e-b418-abe096239476)

Result: detected a Trojan sent via Metasploit.

![image](https://github.com/user-attachments/assets/35958cbe-1531-4be1-9712-85265df2d9b6)

Looked around on this simulated detection platform and found a place to submit malicious hashes:

![image](https://github.com/user-attachments/assets/2e49890c-db40-4bc3-b1b2-4fbcb8746a74)

I submitted the MD5 hash of the first sample and we passed the first detection phase:

![image](https://github.com/user-attachments/assets/c93bcdd6-0879-4868-9835-a511e4d4e524)

### SAMPLE 2: IP ADDRESSES

![image](https://github.com/user-attachments/assets/309c547c-77c0-4ed6-85c1-d75897682dda)

The new malware sample is apparently just slightly changed, but this would be more than enough to result in a different hash, which likely won’t be detected in this scenario.

![image](https://github.com/user-attachments/assets/f1f13241-9c44-4dfd-b6cd-6384061e47f2)

It still picks up the Metasploit tool, and it also detects a connection to an unusual IP.

The port `4444` is commonly associated with reverse shell connections, and the attacker is likely hosting a reverse shell at this URL to which the victim would connect:

![image](https://github.com/user-attachments/assets/b778faae-b677-4aee-8e50-9c65a461cc81)

Since the attacker IP is the destination here, we need to block all outgoing (egress) traffic to this IP:

![image](https://github.com/user-attachments/assets/1832428a-1c66-474b-bc95-c51c018feb6e)

### SAMPLE 3: DOMAINS

This time, the red teamer is using a pool of public IPs, so we can’t use the IP detection method now:

![image](https://github.com/user-attachments/assets/acca88b0-fec9-4aba-b80f-ca00faee335f)

This time, there was a suspicious executable found to have been downloaded from the internet:

![image](https://github.com/user-attachments/assets/8bfb334a-b29a-438c-94f6-6769f5c030fe)

By the name alone, this is designed as a persistence mechanism.

The domain from which the file was downloaded also looks highly suspicious:

![image](https://github.com/user-attachments/assets/c19afc0e-8f10-40bf-b249-8c61e021089a)

Rather than blocking the IP, which is constantly changing now, we can likely just block the domain by creating a DNS rule:

![image](https://github.com/user-attachments/assets/941ee2ef-6bb1-4616-a681-deb4f7fe56cc)

Success:

![image](https://github.com/user-attachments/assets/9b36343c-b318-4a05-8f90-1cc0892def97)

### SAMPLE 4: ARTIFACTS

Blocking the hash, IP, or domain won’t work here, so we have to focus on the artifacts themselves:

![image](https://github.com/user-attachments/assets/4f947e2f-d8a4-4802-a9e0-e01628158cfd)

In the registry activity section of the analysis report, we have a record of a sample4.exe that disables Windows Defender’s real-time protection on the system:

![image](https://github.com/user-attachments/assets/3901029e-1fac-4c03-b9e4-8422ea6c46fa)

With the Sigma Rule Builder tool, it looks like we can create a new detection rule, so I created a new Sysmon rule that detects registry modification of the Real-Time Protection key’s DisableRealtimeMonitoring registry (a value of 1 means that registry is now active):

![image](https://github.com/user-attachments/assets/3b7f4e5b-572e-4509-a44b-57e8b16ffed8)

The MITRE ATT&CK ID is naturally Defense Evasion, since the goal of this modification was to turn off the system’s defense perimeter.

And we managed to tick off the red teamer :)

![image](https://github.com/user-attachments/assets/56e3aab0-7852-4c0f-b4b7-9884cdf959ee)

### SAMPLE 5: TOOLS

In our last stretch, it looks like we have to now focus on any abnormal behaviors of the tool itself:

![image](https://github.com/user-attachments/assets/fffe3832-16f6-4650-915b-082de11a5d46)

Rather than the malware sample, we got an outgoing connection log this time:

![image](https://github.com/user-attachments/assets/3e74d191-2509-422b-837b-5693a5298947)

Two of the lines that stand out show that huge amounts of data are being sent to different IPs, but on the same port 443 (HTTPS). A sign of exfiltration maybe? But it’s also notable that exactly 97 bytes are included in each packet sent to the IP 51.102.10.19. And each of these packets are spaced 30 minutes apart, which may be a sign of beaconing (aka, sending traffic to a C2 server at specific time intervals:

![image](https://github.com/user-attachments/assets/666b4063-d27f-4a45-96f2-798054b57f97)

That’s why it’s important to pay attention to patterns like these. It’s how we separate the normal from abnormal.

On those lines, I created a Sysmon rule for network connections. The first IP I entered was incorrect, and I got a message saying that the attacker had evolved, so I expanded my remote IP and port coverage to “Any” and also set the ATT&CK ID to reflect a C2 (Command and Control) communication:

![image](https://github.com/user-attachments/assets/42faeb35-bcf0-47a3-9bdc-7a4e01d84ed5)

Red teamer is super annoyed now:

![image](https://github.com/user-attachments/assets/3acff26e-d585-4296-a37f-ec8944e7718f)

And that’s the purpose: to get the attacker to give up. But it looks like we still have one more level left.

### SAMPLE 6: TTPs

We need to focus on the attacker “techniques and procedures” at this point, something they have absolutely no control over and can’t change. For this last stage, the red teamer gave us the command logs from all the previous malware samples, so we need to try and understand his methodology:

![image](https://github.com/user-attachments/assets/ac5a5de3-4b56-4218-88af-93baed1d715c)

Red teamer seems to be working on the Command Prompt, appending information about the system to a single file in the Temp directory called `exfiltr8.log`:

![image](https://github.com/user-attachments/assets/5fbffff6-03c5-4a2d-b633-fe4eed257a50)

A quick overview of the information that’s clearly being exfiltrated:

- Directory listings of `C:\`, `C:\Documents and Settings`, `C:\Program Files`, and `D:\`
- `net localgroup administrator`   Member listing of the Administrator group
- `systeminfo`, `ver`   System info, including version of the OS and other info
- `ipconfig /all`   A listing of network adapters info, including IP and MAC addresses
- `netstat -ano`   A listing of active TCP connections, which includes PID info for each connecton
- `net start`   Displays a list of running services

Since we’ve determined this to be a sure sign of exfiltration, this seems to be a simple case of detecting further changes to this log file from the Temp directory, at least in this case:

![image](https://github.com/user-attachments/assets/f3ead951-d7f9-4600-baa6-1d1c99140e6b)

And success!

![image](https://github.com/user-attachments/assets/1222b1c4-13e2-4ab7-b68b-0212a8a688a5)

A few final words from our tenacious red teamer:

![image](https://github.com/user-attachments/assets/c0d8e9ec-33bc-454e-82b5-0ca21063c50d)

That was a rather fun way of visualizing the pyramid of pain!

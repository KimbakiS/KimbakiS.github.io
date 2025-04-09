---
layout: default
---

DHCP is like a network (apartment) landlord, automatically assigning IP addresses (room numbers) to devices (tenants). The problem is that in order to maintain efficiency, it uses the UDP protocol, meaning that it doesn't establish a secure connection with a client before each party transmits data. 

In a 𝐃𝐇𝐂𝐏 𝐬𝐩𝐨𝐨𝐟𝐢𝐧𝐠 𝐚𝐭𝐭𝐚𝐜𝐤, the attacker creates a rogue DHCP server that responds to requests for IPs made by devices on the network before an authentic DHCP server does. As a result, it sends malicious configurations to these devices, which can then open the doors for other attacks.

𝐌𝐚𝐧-𝐢𝐧-𝐭𝐡𝐞-𝐌𝐢𝐝𝐝𝐥𝐞: All traffic that goes outside the network has to pass through the default gateway. If the attacker put an IP address of a device they control as the gateway, that means they would be able to read all the traffic sent back and forth to the victim's device. This is essentially a type of Man-in-the-Middle (or eavesdropping) attack, which is similar to a nosy neighbor (posing as your landlord) receiving your mail...they can potentially sneak a peak before delivering it to you.

𝐃𝐍𝐒 𝐒𝐩𝐨𝐨𝐟𝐢𝐧𝐠: DNS is like a yellow pages phonebook, matching websites with an IP address, and (very simply put) a DNS server manages these records. When an attacker injects a DNS server with fake records, anyone connecting to the server is potentially sent to a fake website other than the one they were expecting. So, the results are self-explanatory when the attacker sends along the compromised DNS server as part of the DHCP configurations.

𝐃𝐇𝐂𝐏 𝐒𝐭𝐚𝐫𝐯𝐚𝐭𝐢𝐨𝐧 𝐀𝐭𝐭𝐚𝐜𝐤: As a common precursor to DHCP spoofing, the attacker floods a legitimate DHCP server with so many requests that it ends up depleting the server's IP address pool. The aim here would be to force clients within the network to connect to the rogue server instead of the legitimate one. It's a more sure-fire and sophisticated method instead of simply placing the rogue server closer to a client and depending on chance.

And these are just a few connections I could think of. It's rather eye-opening, albeit a bit scary, to connect different attacks and see just how they fit into a single picture together.

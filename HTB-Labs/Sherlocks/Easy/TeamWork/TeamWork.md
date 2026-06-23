- **Date**: Thursday, May 14, 2026
- **Username**: cartofelplm
- **Difficulty**: Easy
- **Tags**: #Threat_Intelligence #Email #MITRE #Moonstone_Sleet
- **Scenario**: It is Friday afternoon and the SOC at Edny Consulting Ltd has received alerts from the workstation of Jason Longfield, a software engineer on the development team, regarding the execution of some discovery commands. Jason has just gone on holiday and is not available by phone. The workstation appears to have been switched off, so the only evidence we have at the moment is an export of his mailbox containing today's messages. As the company was recently the victim of a supply chain attack, this case is being taken seriously and the Cyber Threat Intelligence team is being called in to determine the severity of the threat.
- **Sherlock Info**: Starting with a suspicious email, you need to pivot and find as much information as possible about the potential threat actor targeting Jason, collect indicators, and map their techniques to the MITRE ATT&CK framework.
- **Official Writeup**: [here](https://htb-content-prod-private-storage.s3.eu-central-1.amazonaws.com/sherlocks/writeup/9e840d1d-81f9-4268-bc42-628f82d9ca32.pdf?X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIA47CRVXI3GZ5T5FNV%2F20260514%2Feu-central-1%2Fs3%2Faws4_request&X-Amz-Date=20260514T150335Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Signature=54c93d3a34f5f83e830fab7795c1309465cb9f272c7b91aa180a328ac5cea0a1)
- **Video Walkthrough**: none
- **Impression**: interesting

---
# Artefacts

We unzip the `.zip` file provided:

```
└──╼ [*]$ unzip -P hacktheblue TeamWork.zip 
Archive:  TeamWork.zip
   creating: jasonlongfield@edny.net/
  inflating: jasonlongfield@edny.net/SEC Drops Consensys Case ⚖️, Base Upgrades 🦾, Metamask’s Updates 🦊.eml  
  inflating: jasonlongfield@edny.net/Update on JavaScript Authentication Module.eml  
  inflating: jasonlongfield@edny.net/Microsoft will pull the plug on Skype in May.eml  
  inflating: jasonlongfield@edny.net/Your New Online Project Management Tool Smartsheet.eml  
  inflating: jasonlongfield@edny.net/Your online event invitation for "Sync before Jason holiday Software Development Progress and Insights".eml  
  inflating: jasonlongfield@edny.net/49K Building Systems Exposed 🏢, Cellebrite blocks Serbia 📱, Cracking Dashcams 📷.eml  
  inflating: jasonlongfield@edny.net/HashiCorp joins IBM 🤝, Custom Transport Protocol ✨, Copilot for Azure DevOps 🔮.eml  
  inflating: jasonlongfield@edny.net/GPT 4.5 4️⃣, Meta AI Chatbot App 📱, Emergent Misalignment ⚖️.eml  
  inflating: jasonlongfield@edny.net/SWLW #640 The burdens of data, Creating a sense of stability, and more.eml  
  inflating: jasonlongfield@edny.net/A reference manual for people who design and build software.eml  
  inflating: jasonlongfield@edny.net/OpenAI launches GPT-4.5 🧠, Figure home robots 🤖, advice for CS students 👨‍💻.eml  
  inflating: jasonlongfield@edny.net/GibberLink Breakthrough in How Voice Assistants Communicate AI-to-AI.eml  
  inflating: jasonlongfield@edny.net/Opportunity to Invest in NFT Game Project.eml  
```

We see a bunch of `.eml` files that we will analyze during our investigation. These correspond to email messages that are saved on the target system.

---
# Questions

1. Identify the sender of the suspicious email.

**Answer**: `theodore.todtenhaupt@developingdreams.site`

We take a look at the `Opportunity to Invest in NFT Game Project.eml` file and we believe this is a good indication of the suspicious e-mail they referred in this Sherlock's scenario that Jason mostly probably downloaded. The e-mail is suspicious because:
- there is a request for cooperation
- there is a link to download a file
- there is a password to open the file, indicating that it may be an encrypted archive. This is a common technique used to prevent the email filtering systems from scanning attachments.

![[TeamWork-1778771760616.webp]]

2. The suspicious email came from a custom domain, identify its creation date.

**Answer**: 2025-01-31

We use [this](https://who.is/) website to look for registration information about `developingdreams.site`:

![[TeamWork-1778772009421.webp]]

3. The domain was registered shortly before the suspicious email was received, which likely corresponds to the time when the threat actor was planning this campaign. Which MITRE ATT&CK sub-technique of the Resource Development tactic corresponds to this activity?

**Answer**: `T1583.001`

![[TeamWork-1778772518722.webp]]

4. The previously identified domain appears to belong to a company, what is the full URL of the company's page on X (formerly Twitter)?

**Answer**: https://x.com/Develop_Dreams

The website is not longer available, so we access it using the WayBack Machine.

![[TeamWork-1778772975348.webp]]

We access one of the earlier snapshot from the 4th of February 2025 and find the URL we are looking for by hovering over the Twitter logo.

![[TeamWork-1778773079110.webp]]

5. Reading the suspicious email carefully, it appears that the threat actor first contacted the victim using the previously identified social media profile. Which MITRE ATT&CK sub-technique of the Resource Development tactic corresponds to this activity?

**Answer**: T1585.001

![[TeamWork-1778773288296.webp]]

6. What is the name of the game the threat actor would like us to collaborate on?

**Answer**: DeTankWar

We find this info on their website (that once existed):

![[TeamWork-1778773471430.webp]]

7. What is the SHA-256 hash of the executable shared by the threat actor?

**Answer**: 56554117d96d12bd3504ebef2a8f28e790dd1fe583c33ad58ccbf614313ead8c

We click on the `Beta` button and download the `.zip` file from the email. We unzip its contents and compute the SHA-256 of the executable extracted:

```
└──╼ [*]$ sha256sum beta_release_v.1.32.exe 
56554117d96d12bd3504ebef2a8f28e790dd1fe583c33ad58ccbf614313ead8c  beta_release_v.1.32.exe
```

8. As part of the preparation of the tools for the attack, the threat actor hosted this file, presumably malware, on its infrastructure. Which MITRE ATT&CK sub-technique of the Resource Development tactic corresponds to this activity?

**Answer**: T1608.001

![[TeamWork-1778781803223.webp]]

9. Based on the information you have gathered so far, do some research to identify the name of the threat actor who may have carried out this attack.

**Answer**: Moonstone Sleet

We use the SHA-256 value we found before and look it up in VirusTotal. It is indeed detected as dangerous. We access `Community` tab and
find a comment that mentions a report which includes the file we analyzed as an IoC:

![[TeamWork-1778782069302.webp]]

We access the URL provided and find that it belongs to Moonstone Sleet -- a new North Korean threat actor. 

![[TeamWork-1778782130568.webp]]

10. What nation is the threat actor believed to be associated with?

**Answer**: North Korea

See above.

11. Another campaign from this threat actor used a trojanized version of a well-known software to infect victims. What is the name of this tool?

**Answer**: PuTTY

We look for Moonstone Sleet and find this on MITRE:

![[TeamWork-1778782213563.webp]]

12. Which MITRE ATT&CK technique corresponds to the activity of deploying trojanized/manipulated software?

**Answer**: T1195.002

See above.

13. Our company wants to protect itself from other supply chain attacks, so in documenting more about this threat actor, the CTI team found that other security researchers were also tracking a group whose techniques closely match the threat actor we identified, and discovered a new supply chain campaign around the end of July 2024. What technology is this campaign targeting?

**Answer**: npm

We make a query online looking for "moonstone sleet supply chain july 2024". We find [this](https://securitylabs.datadoghq.com/articles/stressed-pungsan-dprk-aligned-threat-actor-leverages-npm-for-initial-access/) article that seems to be what we are looking for:

![[TeamWork-1778782905974.webp]]

14. We now need some indicators to be able to rule out that other systems have been compromised. What is the name and version of the lastest malicious package published? (Format: package-name vX.X.X)

**Answer**: harthat-hash v1.3.3

Reading the article we see that `harthat-hash v1.3.3` is the latest malicious package published.

![[TeamWork-1778782989435.webp]]

15. The malicious packages downloaded an additional payload from a C2 server, what is its IP address?

**Answer**: 142.111.77.196

We find the answer in the article:

![[TeamWork-1778783143030.webp]]

16. The payload, after being renamed, is finally executed by a legitimate Windows binary to evade defenses. Which MITRE ATT&CK technique corresponds to this activity?

**Answer**: T1218.011

In the article it is suggested that the downloaded payload is a DLL loaded into memory using the `rundll32.exe` binary.

![[TeamWork-1778783465863.webp]]

This is an indication of the `System Binary Proxy Execution` technique with `Rundll32` as its subtechnique.

![[TeamWork-1778783535874.webp]]

---
# Tools Used

List of tools/commands used: #whois #wayback_machine #virusTotal 

---

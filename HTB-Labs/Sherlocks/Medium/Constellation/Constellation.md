- **Date**: Tuesday, May 12, 2026
- **Username**: cartofelplm
- **Difficulty**: Medium
- **Tags**: #Threat_Intelligence 
- **Scenario**: The SOC team has recently been alerted to the potential existence of an insider threat. The suspect employee's workstation has been secured and examined. During the memory analysis, the Senior DFIR Analyst succeeded in extracting several intriguing URLs from the memory. These are now provided to you for further analysis to uncover any evidence, such as indications of data exfiltration or contact with malicious entities. Should you discover any information regarding the attacking group or individuals involved, you will collaborate closely with the threat intelligence team. Additionally, you will assist the Forensics team in creating a timeline.
- **Sherlock Info**: This medium-difficulty Sherlock requires a deft application of Open Source Intelligence (OSINT) skills to unravel a convoluted web of deceit and betrayal. As the story unfolds, you are tasked with investigating a suspected insider threat. The scenario begins with the SOC team's recent discovery of potential malicious activities within Forela's organization. A suspect employee's workstation, now secured and thoroughly examined, yields the first clues. Your primary objective is to analyze several unusual URLs extracted from the employee's computer memory by the Senior DFIR Analyst.
- **Official Writeup**: [here](https://htb-content-prod-private-storage.s3.eu-central-1.amazonaws.com/sherlocks/writeup/9e4d9104-e6c2-4417-8436-9081b5ba6005.pdf?X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIA47CRVXI3GZ5T5FNV%2F20260512%2Feu-central-1%2Fs3%2Faws4_request&X-Amz-Date=20260512T173738Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Signature=52332912f5ed7d320133898604a1ad20d7262f29c03943186076e601fa4c9e0f)
- **Video Walkthrough**: none
- **Impression**: really interesting

---
# Artefacts

We are provided with `IOCs.txt` and `NDA_Instructions.pdf`.

---
# Questions

1. When did the suspect first start Direct Message (DM) conversations with the external entity (A possible threat actor group which targets organizations by paying employees to leak sensitive data)? (UTC)

**Answer**: 2023-09-16 16:03:37

![[Constellation-1778609084640.webp]]

2. What was the name of the file sent to the suspected insider threat?

**Answer**: NDA_Instructions.pdf

3. When was the file sent to the suspected insider threat? (UTC)

**Answer**: 2023-09-27 05:27:02

![[Constellation-1778609376707.webp]]

We know this is the correct timestampt because we see the `File ID` entity. 

4. The suspect utilised Google to search something after receiving the file. What was the search query?

**Answer**: how to zip a folder using tar in linux

Just use the second URL provided in `IOCs.txt`.

5. The suspect originally typed something else in search tab, but found a Google search result suggestion which they clicked on. Can you confirm which words were written in search bar by the suspect originally?

**Answer**: How to archive a folder using tar i

We look at the url:

`https://www.google.com/search?q=how+to+zip+a+folder+using+tar+in+linux&sca_esv=568736477&hl=en&sxsrf=AM9HkKkFWLlX_hC63KqDpJwdH9M3JL7LZA%3A1695792705892&source=hp&ei=Qb4TZeL2M9XPxc8PwLa52Ag&iflsig=AO6bgOgAAAAAZRPMUXuGExueXDMxHxU9iRXOL-GQIJZ-&oq=How+to+archive+a+folder+using+tar+i&gs_lp=Egdnd3Mtd2l6IiNIb3cgdG8gYXJjaGl2ZSBhIGZvbGRlciB1c2luZyB0YXIgaSoCCAAyBhAAGBYYHjIIEAAYigUYhgMyCBAAGIoFGIYDMggQABiKBRiGA0jI3QJQ8WlYxIUCcAx4AJABAJgBqQKgAeRWqgEEMi00NrgBAcgBAPgBAagCCsICBxAjGOoCGCfCAgcQIxiKBRgnwgIIEAAYigUYkQLCAgsQABiABBixAxiDAcICCBAAGIAEGLEDwgILEAAYigUYsQMYgwHCAggQABiKBRixA8ICBBAjGCfCAgcQABiKBRhDwgIOEC4YigUYxwEY0QMYkQLCAgUQABiABMICDhAAGIoFGLEDGIMBGJECwgIFEC4YgATCAgoQABiABBgUGIcCwgIFECEYoAHCAgUQABiiBMICBxAhGKABGArCAggQABgWGB4YCg&sclient=gws-wiz`

![[Constellation-1778610106728.webp]]

6. When was this Google search made? (UTC)

**Answer**:

![[Constellation-1778610212213.webp]]

7. What is the name of the Hacker group responsible for bribing the insider threat?

**Answer**: AntiCorp Gr04p

![[Constellation-1778610272706.webp]]

8. What is the name of the person suspected of being an Insider Threat?

**Answer**: karen riley

9. What is the anomalous stated creation date of the file sent to the insider threat? (UTC)

**Answer**: 2054-01-17 21:45:22

```
└──╼ [*]$ exiftool NDA_Instructions.pdf 
ExifTool Version Number         : 13.25
File Name                       : NDA_Instructions.pdf
Directory                       : .
File Size                       : 26 kB
File Modification Date/Time     : 2024:03:05 11:02:19+01:00
File Access Date/Time           : 2026:05:12 20:24:02+02:00
File Inode Change Date/Time     : 2026:05:12 20:19:45+02:00
File Permissions                : -rw-rw-r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.7
Linearized                      : No
Page Count                      : 1
Producer                        : AntiCorp PDF FW
Create Date                     : 2054:01:17 22:45:22+01:00
Title                           : KarenForela_Instructions
Author                          : CyberJunkie@AntiCorp.Gr04p
Creator                         : AntiCorp
Modify Date                     : 2054:01:17 22:45:22+01:00
Subject                         : Forela_Mining stats and data campaign (Stop destroying env)

```

10. The Forela threat intel team are working on uncovering this incident. Any OpSec mistakes made by the attackers are crucial for Forela's security team. Try to help the TI team and confirm the real name of the agent/handler from Anticorp.

**Answer**: Abdullah Al Sajjad

![[Constellation-1778610642078.webp]]

11. Which City does the threat actor belong to?

**Answer**: Bahawalpur

---
# Tools Used

List of tools used: #unfurl #exiftool

---

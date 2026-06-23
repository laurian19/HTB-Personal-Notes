- **Date**: Thursday, May 14, 2026
- **Username**: cartofelplm
- **Difficulty**: Easy
- **Tags**: #SOC #Brute-force 
- **Scenario**: As a fast-growing startup, Forela has been utilising a business management platform. Unfortunately, our documentation is scarce, and our administrators aren't the most security aware. As our new security provider we'd like you to have a look at some PCAP and log data we have exported to confirm if we have (or have not) been compromised.
- **Sherlock Info**: You are brought in as the new security provider for Forela, a fast-growing startup. The startup has been using a business management platform, but with insufficient documentation and potentially lax security practices. You are provided with PCAP and log data and are tasked with determining if a compromise has occurred. This scenario pushes the you to employ your analysis skills, effectively sifting through network data and logs to detect potential signs of intrusion, thereby offering a realistic taste of the pivotal role cybersecurity plays in the protection of burgeoning businesses.
- **Official Writeup**: [here](https://htb-content-prod-private-storage.s3.eu-central-1.amazonaws.com/sherlocks/writeup/9e4d9105-f4e8-4a42-a057-fa8852eb142a.pdf?X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIA47CRVXI3GZ5T5FNV%2F20260514%2Feu-central-1%2Fs3%2Faws4_request&X-Amz-Date=20260514T212117Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Signature=3a5adb30a3f17f8a3dfcc2338b05678f842c30da13520308583c51cd0a0a90b4)
- **Video Walkthrough**: none
- **Impression**:

---
# Artefacts

We unzip the provided `.zip` file:

```
└──╼ [*]$ unzip -P hacktheblue meerkat.zip 
Archive:  meerkat.zip
  inflating: meerkat.pcap            
  inflating: meerkat-alerts.json     
```

As specified in the Sherlock's scenario, we will analyze a `.pcap` file and a log file to determine whether the business management platform has been compromised or not.

---
# Questions

1. We believe our Business Management Platform server has been compromised. Please can you confirm the name of the application running?

**Answer**: Bonitasoft

We take a look at the logs provided first. Here is the first entry in the log file:

```
└──╼ [*]$ jq '.[0]' meerkat-alerts.json  
{
  "ts": "2023-01-19T15:44:49.669971Z",
  "event_type": "alert",
  "src_ip": "89.248.165.187",
  "src_port": 52870,
  "dest_ip": "172.31.6.44",
  "dest_port": 10227,
  "vlan": null,
  "proto": "TCP",
  "app_proto": null,
  "alert": {
    "severity": 2,
    "signature": "ET CINS Active Threat Intelligence Poor Reputation IP group 82",
    "category": "Misc Attack",
    "action": "allowed",
    "signature_id": 2403381,
    "gid": 1,
    "rev": 80387,
    "metadata": {
      "signature_severity": [
        "Major"
      ],
      "former_category": null,
      "attack_target": [
        "Any"
      ],
      "deployment": [
        "Perimeter"
      ],
      "affected_product": [
        "Any"
      ],
      "created_at": [
        "2013_10_08"
      ],
      "performance_impact": null,
      "updated_at": [
        "2023_01_18"
      ],
      "malware_family": null,
      "tag": [
        "CINS"
      ]
    }
  },
  "flow_id": 519087154346259,
  "pcap_cnt": 6292,
  "tx_id": null,
  "icmp_code": null,
  "icmp_type": null,
  "tunnel": null,
  "community_id": "1:+BXi7peXaBKuiEO4y3Ya0UlQMMQ="
}
```

We believe the `signature` field can give us more information about the application being ran:

```
└──╼ [*]$ jq '.[].alert.signature' meerkat-alerts.json | sort | uniq
"ET 3CORESec Poor Reputation IP group 18"
"ET 3CORESec Poor Reputation IP group 42"
"ET ATTACK_RESPONSE Possible /etc/passwd via HTTP (linux style)"
"ET CINS Active Threat Intelligence Poor Reputation IP group 13"
"ET CINS Active Threat Intelligence Poor Reputation IP group 29"
"ET CINS Active Threat Intelligence Poor Reputation IP group 31"
"ET CINS Active Threat Intelligence Poor Reputation IP group 76"
"ET CINS Active Threat Intelligence Poor Reputation IP group 81"
"ET CINS Active Threat Intelligence Poor Reputation IP group 82"
"ET CINS Active Threat Intelligence Poor Reputation IP group 84"
"ET DROP Dshield Block Listed Source group 1"
"ET EXPLOIT Bonitasoft Authorization Bypass M1 (CVE-2022-25237)"
"ET EXPLOIT Bonitasoft Authorization Bypass and RCE Upload M1 (CVE-2022-25237)"
"ET EXPLOIT Bonitasoft Successful Default User Login Attempt (Possible Staging for CVE-2022-25237)"
"ET INFO User-Agent (python-requests) Inbound to Webserver"
"ET POLICY GNU/Linux APT User-Agent Outbound likely related to package management"
"ET SCAN Potential VNC Scan 5800-5820"
"ET SCAN Potential VNC Scan 5900-5920"
"ET SCAN Suspicious inbound to MSSQL port 1433"
"ET SCAN Suspicious inbound to Oracle SQL port 1521"
"ET SCAN Suspicious inbound to PostgreSQL port 5432"
"ET SCAN Suspicious inbound to mySQL port 3306"
"ET WEB_SPECIFIC_APPS Bonitasoft Default User Login Attempt M1 (Possible Staging for CVE-2022-25237)"
"GPL SNMP public access udp"
"GPL WEB_SERVER DELETE attempt"
null
```

We find the name of the software, which is `Bonitasoft`.

2. We believe the attacker may have used a subset of the brute forcing attack category - what is the name of the attack carried out?

**Answer**:

We take a look at the `.pcap` file provided (using Wireshark) and filter for `http` traffic only:

![[Meerkat-1778795190439.webp]]

We see multiple login attempts to the `/bonita/loginservice` page using different credentials. We apply the following display filter `http.request.uri contains loginservice and http.user.agent == python-requests/2.28.1` and this is the content of one of the login attempts: 

![[Meerkat-1778795244298.webp]]

This is an indication of a brute-force attack. We want to determine each username-password combination used. For that, we will use `tshark`:

```
tshark -Y (http.request.uri contains loginservice and http.user.agent == python-requests/2.28.1) -r meerk
```


3. Does the vulnerability exploited have a CVE assigned - and if so, which one?

**Answer**:

4. Which string was appended to the API URL path to bypass the authorization filter by the attacker's exploit?

**Answer**:

5. How many combinations of usernames and passwords were used in the credential stuffing attack?

**Answer**:

6. Which username and password combination was successful?

**Answer**:

7. If any, which text sharing site did the attacker utilise?

**Answer**:

8. Please provide the filename of the public key used by the attacker to gain persistence on our host.

**Answer**:

9. Can you confirm the file modified by the attacker to gain persistence?

**Answer**:

10. Can you confirm the MITRE technique ID of this type of persistence mechanism?

**Answer**:

---
# Tools Used

List of tools/commands used: #wireshark #json

---

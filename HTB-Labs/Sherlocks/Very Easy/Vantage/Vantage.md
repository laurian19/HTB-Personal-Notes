- **Date**: Thursday, May 14, 2026
- **Username**: cartofelplm
- **Difficulty**: Very Easy
- **Tags**: #DFIR #Fuzzing #OpenStack #Keystone #MITRE 
- **Scenario**: A small company moved some of its resources to a private cloud installation. The developers left the redirect to the dashboard on their web server. The security team got an email from the alleged attacker stating that the user data was leaked. It is up to you to investigate the situation.
- **Sherlock Info**: none yet -- still acitve
- **Official Writeup**: none
- **Video Walkthrough**: none
- **Impression**: very very cool sherlock

---
# Artefacts

We unzip the `.zip` file provided:

```
└──╼ [*]$ unzip -P hacktheblue Vantage.zip 
Archive:  Vantage.zip
   creating: Vantage/
  inflating: Vantage/.DS_Store       
  inflating: Vantage/controller.2025-07-01.pcap  
  inflating: Vantage/web-server.2025-07-01.pcap  
```

We will analyze two `.pcap` files later on, using Wireshark. We take a look at the type of the other file:

```
└──╼ [*]$ file .DS_Store 
.DS_Store: Apple Desktop Services Store
```

It looks like `.DS_Store` is a file generated automatically by macOS that stores custom attributes of the containing folder, such as folder view options, icon positions, and other visual information. My feeling right now is that we won't need this file later on within our analysis.

---
# To Know

Read [this](https://www.zenarmor.com/docs/network-basics/what-is-openstack-api) about OpenStack. 21

---
# Questions

1. What tool did the attacker use to fuzz the web server ? (Format- include version e.g, [nmap@7.80](mailto:nmap@7.80))

**Answer**: ffuf@2.1.0

We open `web-server.2025-07-01.pcap` in Wireshark and look for protocols available:

![[Vantage-1778749292037.webp]]

We will first take a look at HTTP, so we apply the Wireshark filter `http` and follow the HTTP stream from the first packet:

![[Vantage-1778749343559.webp]]

This is enough information to determine the they used the FFUF (Fuzz Faster You Full) fuzzer with version 2.1.0.

2. Which subdomain did the attacker discover?

**Answer**: cloud

We are looking at subdomain fuzzing/virtual host fuzzing, where an attacker is sending multiple HTTP requests to the same server, but change the `Host` header. 

The idea is to look for the HTTP request(s) that do not return default responses, such as the following:

```
HTTP/1.1 200 OK

Date: Tue, 01 Jul 2025 09:38:37 GMT

Server: Apache/2.4.58 (Ubuntu)

Last-Modified: Fri, 27 Jun 2025 05:25:15 GMT

ETag: "29af-63886e72d0219-gzip"

Accept-Ranges: bytes

Vary: Accept-Encoding

Content-Encoding: gzip

Content-Length: 3121

Content-Type: text/html

  

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml">

<!--

Modified from the Debian original for Ubuntu

Last updated: 2022-03-22

See: https://launchpad.net/bugs/1966004

-->

<head>

<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

<title>Apache2 Ubuntu Default Page: It works</title>

<style type="text/css" media="screen">

* {

margin: 0px 0px 0px 0px;

padding: 0px 0px 0px 0px;

}

  

body, html {

padding: 3px 3px 3px 3px;

  

background-color: #D8DBE2;

  

font-family: Ubuntu, Verdana, sans-serif;

font-size: 11pt;

text-align: center;

}

  

div.main_page {

position: relative;

display: table;

...
```

We include the following as columns in Wireshark `http.host` and `http.response.code`. 

We see that packet 20666 was an HTTP request to `/` on the host `cloud.vantage.tech` and has the following HTTP response:

![[Vantage-1778752264173.webp]]

This is a bingo, as it is an indicator that the attacker found the subdomain `cloud` of host `vantage.tech`.

3. How many login attempts did the attacker make before successfully logging in to the dashboard?

**Answer**: 3

We use the following filter `http.request.uri contains login and ip.addr == 117.200.21.26` to look only for the login attempts initiated by the attacker.

![[Vantage-1778754402547.webp]]

There are 4 login attempts but it seems like the 4th one is successful. The corresponding credentials are `admin:StrongAdminSecret`.

4. When did the attacker download the OpenStack API remote access config file? (UTC)

**Answer**: 2025-07-01 09:40:29

![[Vantage-1778754733919.webp]]

5. When did the attacker first interact with the API on controller node? (UTC)

**Answer**: 2025-07-01 09:41:44

We open the other `.pcap` file in Wireshark. We use the following filter `ip.addr==117.200.21.26` as we want to see when was the first request from the attacker IP address.

![[Vantage-1778754860579.webp]]

6. What is the project id of the default project accessed by the attacker?

**Answer**: 9fb84977ff7c4a0baf0d5dbb57e235c7

We follow the HTTP streams and we find the following stream with the ID of the default project accessed by the attacker:

![[Vantage-1778755089202.webp]]

7. Which OpenStack service provides authentication and authorization for the OpenStack API?

**Answer**: Keystone

We look at the first POST request which includes the `auth` keyword in the request URI and notice that `keystone` is part of the `User-Agent` value.

![[Vantage-1778755415960.webp]]

We confirm our findings online:

![[Vantage-1778755521651.webp]]

8. What is the endpoint URL of the swift service?

**Answer**: `http://134.209.71.220:8080/v1/AUTH_9fb84977ff7c4a0baf0d5dbb57e235c7`

We look through the HTTP streams and find the following request to `/identity/v3/services` and the corresponding response:

![[Vantage-1778765704662.webp]]

We find the Swift service with ID `f9194820052d4788b09157bf0a0dfdd0`. 

In HTTP stream `375`, the attacker created an HTTP GET request to `/identity/v3/endpoints`. The corresponding response includes:

![[Vantage-1778765850188.webp]]

This allows the attacker to determine the Endpoint URL of the Swift service, which is `http://134.209.71.220:8080/v1/AUTH_9fb84977ff7c4a0baf0d5dbb57e235c7`. Note that we use the project default ID and not the Swift service ID.

9. How many containers were discovered by the attacker?

**Answer**: 3

We use the following filter in Wireshark:

```
http and ip.addr == 117.200.21.26 and http.request.uri contains 9fb84977ff7c4a0baf0d5dbb57e235c7
```

![[Vantage-1778767354260.webp]]

We take a look at the first HTTP stream and find out that there are three containers on the Swift service, namely `dev-files`, `employee-data`, and `user-data`.

![[Vantage-1778767395649.webp]]

10. When did the attacker download the sensitive user data file? (UTC)

**Answer**: 2025-07-01 09:45:23

We use the following filter in Wireshark:

```
http and ip.addr == 117.200.21.26 and http.request.uri contains user-data
```

![[Vantage-1778767457854.webp]]

We see the attacker downloading `user-details.csv` from `user-data` container at `2025-07-01 09:45:23`.

11. How many user records are in the sensitive user data file?

**Answer**: 28

We export the `user-details.csv` file from Wireshark -> Export Objects:

![[Vantage-1778767647483.webp]]

Then, we open the `.csv` file and find the answer.

![[Vantage-1778767685188.webp]]

12. For persistence, the attacker created a new user with admin privileges. What is the username of the new user?

**Answer**: jellibean

We use the following filter in Wireshark:

```
http and ip.addr == 117.200.21.26 and http.request.uri contains user
```

![[Vantage-1778767773871.webp]]

And follow the HTTP stream from the highlighted packet:

![[Vantage-1778767799165.webp]]

It looks like the attacker created a new user with name `jellibean` and password `P@$$word` on the controller node, which has access to the default project ID we found beforehand. 

13. What is the password of the new user?

**Answer**: `P@$$word`

See above.

14. What is MITRE tactic id of the technique in task 12?

**Answer**: T1136.003

![[Vantage-1778768033783.webp]]

---
# Tools Used

List of tools/commands used: #wireshark

---
# Bigger Picture

#### Issue

The target company moved some resources into a private OpenStack cloud. However, the developers of the company accidentally left a redirect to the cloud dashboard exposed on the public server. 

#### Attack

The attacker first performed subdomain fuzzing against `vantage.tech` domain. They used `ffuf@2.1.0` and sent many HTTP requests with different `Host` headers. Most of them returned the default Apache page with response code 200, but one hostname behaved differently. This was `cloud.vantage.tech`. A request with this value for the `Host` header did not return the default Apache page, but a redirect to the dashboard login page. At this point, the attacker found the valid cloud dashboard subdomain, which was `cloud`.

Then, they started trying to login. In the telemetry network traffic from the exposed server, there were four login POST requests from the attacker IP `117.200.21.26`. The first three attempts failed, and the last one with credentials `admin:StrongAdminSecret` succeeded. 

Once logged in to the dashboard, the attacker downloaded the OpenStack API remote access configuration file at `2025-07-01 09:40:29 UTC`. This file gave them information needed to interact directly with the OpenStack API via their software, instead of only via the dashboard. 

On the telemetry network from the controlled node, we observe the attacker first interacting with the API on the controller node at `2025-07-01 09:41:44 UTC` (one minute after they downloaded the configuration file). 

The attacker then accesses the default OpenStack project on that controller node, with ID `9fb84977ff7c4a0baf0d5dbb57e235c7`. 

The attacker authenticated to the OpenStack API. The service responsible for authentication and authorization is Keystone. This is important because Keystone gives API tokens that can be used to access other OpenStack services, such as compute, networking, object storage, and others.

After authenticating, the attacker queried the OpenStack service catalog to identify the services available. One of those was Swift, which is OpenStack's object storage service. The attacker found the Swift endpoint: `http://134.209.71.220:8080/v1/AUTH_9fb84977ff7c4a0baf0d5dbb57e235c7`, which includes the default project ID => the attacker is able to access objects stored and linked to that project.

The attacker the interacted with Swift and listed the available containers. They discovered three containers:

- `dev-files`  
- `employee-data`  
- `user-data`

It seems like the most important container was `user-data`, because it contained the sensitive file `user-details.csv`.

At `2025-07-01 09:45:23 UTC`, the attacker downloaded `user-details.csv` from the `user-data` container. After exporting the file from Wireshark and opening it, we found that it contained 28 user records.

After stealing the data, the attacker also created a new OpenStack user for persistence. The new user was named `jellibean`, and the password was `P@$$word`. This account had admin privileges, meaning the attacker could return later without needing to reuse the original `admin` account.

At this point, the full attack chain is clear: the attacker discovered an exposed cloud dashboard, logged in with valid admin credentials, downloaded the OpenStack API configuration, used Keystone to authenticate to the API, enumerated Swift object storage, downloaded sensitive user data, and finally created a new admin user to maintain access.

#### Summary with MITRE TTPs

1. Attacker fuzzed the public web server.  
   → `T1595.003 - Active Scanning: Wordlist Scanning`

2. Attacker discovered the cloud dashboard at `cloud.vantage.tech`.  
   → `T1595.003 - Active Scanning: Wordlist Scanning`

3. Attacker attempted to log in several times.  
   → `T1110.001 - Brute Force: Password Guessing`

4. After 3 failed attempts, the attacker successfully logged in as admin.  
   → `T1078.004 - Valid Accounts: Cloud Accounts`

5. Attacker downloaded the OpenStack API config file.  
   → `T1552 - Unsecured Credentials`

6. Attacker used that config to interact directly with the OpenStack controller API.  
   → `T1078.004 - Valid Accounts: Cloud Accounts`

7. Attacker authenticated through Keystone.  
   → `T1078.004 - Valid Accounts: Cloud Accounts`

8. Attacker accessed the default OpenStack project.  
   → `T1526 - Cloud Service Discovery`

9. Attacker identified the Swift object storage endpoint.  
   → `T1526 - Cloud Service Discovery`

10. Attacker discovered 3 Swift containers: `dev-files`, `employee-data`, and `user-data`.  
   → `T1619 - Cloud Storage Object Discovery`

11. Attacker downloaded `user-details.csv` from the `user-data` container.  
   → `T1530 - Data from Cloud Storage`

12. The stolen file contained 28 user records.  
   → `T1530 - Data from Cloud Storage`

13. Attacker created a new admin user named `jellibean` for persistence.  
   → `T1136.003 - Create Account: Cloud Account`

---

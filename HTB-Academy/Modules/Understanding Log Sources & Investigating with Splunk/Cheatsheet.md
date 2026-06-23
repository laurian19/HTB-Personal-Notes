
---

## 1. Search Basics

|Item|Type|What it does|Example|
|---|---|---|---|
|`search`|command|Explicit search command (usually implicit)|`search index="main" "UNKNOWN"`|
|`index=`|search term|Limits search to an index|`index="main"`|
|`sourcetype=`|search term|Limits search to a sourcetype|`sourcetype="WinEventLog:Sysmon"`|
|`source=`|search term|Limits search to a source|`source="WinEventLog:Sysmon"`|
|`host=`|search term|Limits search to a host|`host="DESKTOP-..."`|
|`EventCode=`|field filter|Filters by event ID/code|`EventCode=1`|
|`field=value`|comparison|Exact field match|`Account_Name="SYSTEM"`|
|`field!=value`|comparison|Exclude a field value|`EventCode!=1`|
|`field<value` / `field>value`|comparison|Numeric comparisons|`Access_Mask=0x100`|
|`*`|wildcard|Matches any number of characters|`Image="*SharpHound.exe"`|
|`AND`|boolean operator|Both conditions must match|`EventCode=1 AND Image="*cmd.exe"`|
|`OR`|boolean operator|Either condition can match|`EventCode=1 OR EventCode=3`|
|`NOT`|boolean operator|Excludes matching results|`NOT [ subsearch ]`|
|`earliest=`|time modifier|Start of search time range|`earliest=-7d`|
|`latest=`|time modifier|End of search time range|`latest=now`|
|`earliest=0`|time modifier|All time (no lower bound)|`index="main" earliest=0`|

---

## 2. Output / Formatting Commands

|Command|What it does|Example|
|---|---|---|
|`table`|Shows only selected fields in table format|`\| table _time, host, Image`|
|`table *`|Shows all available fields|`\| table *`|
|`table _raw`|Shows raw event data|`\| table _raw`|
|`fields`|Includes or excludes fields|`\| fields - User`|
|`rename`|Renames a field|`\| rename Image as Process`|
|`sort`|Sorts results (use `-` for descending)|`\| sort - count`|
|`dedup`|Removes duplicate values/events|`\| dedup Image`|
|`dedup field1, field2`|Deduplicates on multiple fields|`\| dedup filename, is_malware`|

---

## 3. Aggregation & Statistics Commands

| Command                                                                | What it does                                                | Example                                                                                                     |
| ---------------------------------------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `stats count by field`                                                 | Counts events grouped by a field                            | `\| stats count by sourcetype`                                                                              |
| `stats count by f1, f2`                                                | Counts by multiple fields                                   | `\| stats count by ParentImage, Image`                                                                      |
| `stats dc(field)`                                                      | Distinct count of a field's values                          | `\| stats dc(SourceImage) by SourceImage`                                                                   |
| `stats range(_time) as X`                                              | Time range between first and last event                     | `\| stats range(_time) as TimeRange by Account_Name`                                                        |
| `stats values(field) AS alias`                                         | Collects all unique values into a list                      | `\| stats values(Destination) AS "Destinations" by Image`                                                   |
| `stats avg(X) as avg stdev(X) as stdev`                                | Compute average and standard deviation                      | `\| stats avg(threadCount) as avg stdev(threadCount) as stdev`                                              |
| `eventstats avg(X) as avg stdev(X) as stdev`                           | Same as stats but keeps original events                     | `\| eventstats avg(cmdCount) as avg stdev(cmdCount) as stdev`                                               |
| `chart count by f1, f2`                                                | Creates a pivoted table (each value of f2 becomes a column) | `\| chart count by _time, Image`                                                                            |
| `top limit=N field`                                                    | Returns the N most common values                            | `\| top limit=100 Image`                                                                                    |
| `rare limit=N field`                                                   | Returns the N rarest values                                 | `\| rare limit=20 useother=f ParentImage`                                                                   |
| `bucket _time span=1h`                                                 | Groups events into time buckets                             | `\| bucket _time span=1h`                                                                                   |
| `bin _time span=1h`                                                    | Alias for `bucket`                                          | `\| bin _time span=1h`                                                                                      |
| `streamstats time_window=24h avg(X) as avg stdev(X) as stdev by field` | Rolling average/stdev over a sliding window                 | `\| streamstats time_window=24h avg(NetworkConnections) as avg stdev(NetworkConnections) as stdev by Image` |
| `transaction field1, field2`                                           | Groups related events into transactions                     | `\| transaction ComputerName, Image`                                                                        |
| `transaction ... startswith=eval(...) endswith=eval(...) maxspan=1m`   | Transaction with start/end conditions and time limit        | `\| transaction Image startswith=eval(EventCode=1) endswith=eval(EventCode=3) maxspan=1m`                   |

---

## 4. Data Transformation Commands

|Command|What it does|Example|
|---|---|---|
|`eval field=expression`|Creates or redefines a field|`\| eval Process_Path=lower(Image)`|
|`eval field=lower(X)`|Converts a field to lowercase|`\| eval filename=lower(filename)`|
|`eval field=len(X)`|Computes the length of a field|`\| eval len=len(CommandLine)`|
|`eval field=if(condition, val_true, val_false)`|Conditional assignment|`\| eval isOutlier=if(threadCount > avg+2*stdev, 1, 0)`|
|`eval field=mvdedup(split(X, "\\"))`|Splits a path by `\` and deduplicates|`\| eval filename=mvdedup(split(Image, "\\"))`|
|`eval field=mvindex(X, -1)`|Gets last element of a multivalue field|`\| eval filename=mvindex(filename, -1)`|
|`eval field=coalesce(f1, f2)`|Returns first non-null value|`\| eval Destination=coalesce(dest_host,dest_ip)`|
|`rex "(?<name>pattern)"`|Extracts a named group using regex|`\| rex field=Image "(?P<filename>[^\\\]+)$"`|
|`rex max_match=0 "pattern"`|Extracts all matches (not just the first)|`\| rex max_match=0 "[^%](?<guid>{.*})"`|
|`lookup file.csv field OUTPUTNEW new_field`|Enriches events using a CSV lookup table|`\| lookup malware_lookup.csv filename OUTPUTNEW is_malware`|
|`inputlookup file.csv`|Retrieves all records from a lookup file directly|`\| inputlookup malware_lookup.csv`|
|`regex Image="pattern"`|Filters events using a full regex on a field|`\| regex Image="C:\\\\Users\\\\.*\\\\Downloads\\\\.*"`|
|`where condition`|Filters results using an expression|`\| where SourceImage!=TargetImage`|
|`where mvcount(field) > N`|Filters based on multivalue field count|`\| where mvcount(ProcessGuid) > 1`|
|`where TimeRange <= 600`|Numeric comparison on computed field|`\| where TimeRange <= 600`|
|`search isOutlier=1`|Filters inline after stats/eval|`\| search isOutlier=1`|

---

## 5. Data Discovery Commands

| Command                                               | What it does                                  | Example                                                                      |
| ----------------------------------------------------- | --------------------------------------------- | ---------------------------------------------------------------------------- |
| `\| eventcount summarize=false index=*`               | Counts events per index                       | `\| eventcount summarize=false index=* \| table index`                       |
| `\| metadata type=sourcetypes`                        | Lists all sourcetypes with metadata           | `\| metadata type=sourcetypes`                                               |
| `\| metadata type=sourcetypes index=*`                | Lists all sourcetypes with table view         | `\| metadata type=sourcetypes index=* \| table sourcetype`                   |
| `\| metadata type=sources index=*`                    | Lists all data sources                        | `\| metadata type=sources index=* \| table source`                           |
| `\| fieldsummary`                                     | Summarises all fields in result set           | `sourcetype="WinEventLog:Security" \| fieldsummary`                          |
| `\| fieldsummary \| where count < 100`                | Lists rare fields only                        | `\| fieldsummary \| where count < 100 \| table field, count, distinct_count` |
| `\| sistats count by index, sourcetype, source, host` | Counts events by index/sourcetype/source/host | `index=* \| sistats count by index, sourcetype, source, host`                |
| `\| rare limit=10 index, sourcetype`                  | Finds rarest index+sourcetype combinations    | `index=* sourcetype=* \| rare limit=10 index, sourcetype`                    |
| `\| rare limit=10 field1, field2, field3`             | Finds rarest field value combinations         | `\| rare limit=10 field1, field2, field3`                                    |
| `\| stats count by EventCode`                         | Lists all EventCodes present in data          | `index="main" sourcetype="WinEventLog:Sysmon" \| stats count by EventCode`   |
| `\| stats count by sourcetype`                        | Lists sourcetypes with event counts           | `index="main" \| stats count by sourcetype`                                  |

---

## 6. Subsearches

A subsearch is a search nested inside `[ ]` in the outer search.

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
NOT [ search index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
      | top limit=100 Image
      | fields Image ]
| table _time, Image, CommandLine, User, ComputerName
```

---

## 7. Full Queries Used

### 7.1 Basic Searching & Fundamentals

```spl
search index="main" "UNKNOWN"
```

```spl
index="main" "*UNKNOWN*"
```

```spl
index="main" EventCode!=1
```

```spl
index="main" earliest=-7d EventCode!=1
```

```spl
index="main" earliest=0
```

---

### 7.2 Fields, Table, Rename, Dedup, Sort

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | fields - User
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | table _time, host, Image
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | rename Image as Process
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | dedup Image
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | sort - _time
```

---

### 7.3 Stats, Chart, Eval, Rex

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=3 | stats count by _time, Image
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=3 | chart count by _time, Image
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | eval Process_Path=lower(Image)
```

```spl
index="main" EventCode=4662 | rex max_match=0 "[^%](?<guid>{.*})" | table guid
```

---

### 7.4 Lookup

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| rex field=Image "(?P<filename>[^\\\]+)$"
| eval filename=lower(filename)
| lookup malware_lookup.csv filename OUTPUTNEW is_malware
| table filename, is_malware
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| eval filename=mvdedup(split(Image, "\\"))
| eval filename=mvindex(filename, -1)
| eval filename=lower(filename)
| lookup malware_lookup.csv filename OUTPUTNEW is_malware
| table filename, is_malware
| dedup filename, is_malware
```

```spl
| inputlookup malware_lookup.csv
```

---

### 7.5 Transaction & Subsearch

```spl
index="main" sourcetype="WinEventLog:Sysmon" (EventCode=1 OR EventCode=3)
| transaction Image startswith=eval(EventCode=1) endswith=eval(EventCode=3) maxspan=1m
| table Image
| dedup Image
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
NOT [ search index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
      | top limit=100 Image
      | fields Image ]
| table _time, Image, CommandLine, User, ComputerName
```

---

### 7.6 Data Discovery

```spl
| eventcount summarize=false index=* | table index
```

```spl
| metadata type=sourcetypes
```

```spl
| metadata type=sourcetypes index=* | table sourcetype
```

```spl
| metadata type=sources index=* | table source
```

```spl
sourcetype="WinEventLog:Security" | table _raw
```

```spl
sourcetype="WinEventLog:Security" | table *
```

```spl
sourcetype="WinEventLog:Security" | fields Account_Name, EventCode | table Account_Name, EventCode
```

```spl
sourcetype="WinEventLog:Security" | fieldsummary
```

```spl
index=* sourcetype=* | bucket _time span=1d | stats count by _time, index, sourcetype | sort - _time
```

```spl
index=* sourcetype=* | rare limit=10 index, sourcetype
```

```spl
index="main" | rare limit=20 useother=f ParentImage
```

```spl
index=* sourcetype=* | fieldsummary | where count < 100 | table field, count, distinct_count
```

```spl
index=* | sistats count by index, sourcetype, source, host
```

```spl
index=* sourcetype=* | rare limit=10 field1, field2, field3
```

---

### 7.7 Searching Effectively (Performance Tips)

```spl
index="main" | stats count by sourcetype
```

```spl
index="main" sourcetype="WinEventLog:Sysmon"
```

```spl
index="main" uniwaldo.local
```

```spl
index="main" *uniwaldo.local*
```

```spl
index="main" ComputerName="*uniwaldo.local"
```

---

### 7.8 Sysmon EventCode Overview Queries

```spl
index="main" sourcetype="WinEventLog:Sysmon" | stats count by EventCode
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | stats count by ParentImage, Image
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 (Image="*cmd.exe" OR Image="*powershell.exe")
| stats count by ParentImage, Image
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 (Image="*cmd.exe" OR Image="*powershell.exe")
ParentImage="C:\\Windows\\System32\\notepad.exe"
```

---

### 7.9 IP & Host Investigation

```spl
index="main" 10.0.0.229 | stats count by sourcetype
```

```spl
index="main" 10.0.0.229 sourcetype="linux:syslog"
```

```spl
index="main" 10.0.0.229 sourcetype="WinEventLog:sysmon" | stats count by CommandLine
```

```spl
index="main" 10.0.0.229 sourcetype="WinEventLog:sysmon" | stats count by CommandLine, host
```

---

### 7.10 DCSync Detection (EventCode 4662)

```spl
index="main" EventCode=4662 Access_Mask=0x100 Account_Name!=*$
```

---

### 7.11 LSASS Dump Detection (EventCode 10)

```spl
index="main" EventCode=10 lsass | stats count by SourceImage
```

```spl
index="main" EventCode=10 lsass SourceImage="C:\\Windows\\System32\\notepad.exe"
```

```spl
index="main" EventCode=10 lsass SourceImage="C:\\Windows\\system32\\rundll32.exe"
| table CallTrace
| dedup CallTrace
```

---

### 7.12 UNKNOWN Memory Region / Shellcode Alert (EventCode 10)

```spl
index="main" CallTrace="*UNKNOWN*" | stats count by EventCode
```

```spl
index="main" CallTrace="*UNKNOWN*" | stats count by SourceImage
```

```spl
index="main" CallTrace="*UNKNOWN*" | where SourceImage!=TargetImage | stats count by SourceImage
```

```spl
index="main" CallTrace="*UNKNOWN*" SourceImage!="*Microsoft.NET*" CallTrace!=*ni.dll* CallTrace!=*clr.dll*
| where SourceImage!=TargetImage | stats count by SourceImage
```

```spl
index="main" CallTrace="*UNKNOWN*" SourceImage!="*Microsoft.NET*" CallTrace!=*ni.dll* CallTrace!=*clr.dll*
CallTrace!=*wow64* | where SourceImage!=TargetImage | stats count by SourceImage
```

```spl
index="main" CallTrace="*UNKNOWN*" SourceImage!="*Microsoft.NET*" CallTrace!=*ni.dll* CallTrace!=*clr.dll*
CallTrace!=*wow64* SourceImage!="C:\\Windows\\Explorer.EXE"
| where SourceImage!=TargetImage | stats count by SourceImage
```

```spl
index="main" CallTrace="*UNKNOWN*" SourceImage!="*Microsoft.NET*" CallTrace!=*ni.dll* CallTrace!=*clr.dll*
CallTrace!=*wow64* SourceImage!="C:\\Windows\\Explorer.EXE"
| where SourceImage!=TargetImage
| stats count by SourceImage, TargetImage, CallTrace
```

---

### 7.13 C# Injection / Execute-Assembly (EventCode 7 - Image Loaded)

```spl
index="main" EventCode=7 ImageLoaded="*clr.dll"
| stats count by Image
```

```spl
index="main" ParentImage="C:\\Windows\\System32\\rundll32.exe"
| stats count by Image, EventCode, CommandLine
```

---

### 7.14 Exercise Queries

**Kerberos auth ticket requests (EventCode 4768)**

```spl
index="*" EventCode=4768
| stats count by Account_Name
| table Account_Name, count
| sort - count
```

**Distinct computers accessed by SYSTEM (EventCode 4624)**

```spl
index="*" EventCode=4624 Account_Name="SYSTEM"
| fields ComputerName
| dedup ComputerName
```

**Accounts with all login activity within < 10 minutes (EventCode 4624)**

```spl
index=* EventCode=4624
| stats count, range(_time) as TimeRange by Account_Name
| where TimeRange <= 600
| sort -count
```

**Threads created in rundll32 (EventCode 8)**

```spl
index="main" EventCode=8 TargetImage="*rundll32.exe"
| stats dc(SourceImage) by SourceImage
```

**Thread injection outliers (EventCode 8)**

```spl
index="main" EventCode=8 SourceImage!=TargetImage
| bucket _time span=1h
| stats count as threadCount by _time SourceImage
| eventstats avg(threadCount) as avg stdev(threadCount) as stdev
| eval isOutlier=if(threadCount > avg+2*stdev, 1, 0)
| search isOutlier=1
| sort - threadCount
```

**C2 callback IP discovery (EventCode 3)**

```spl
index="main" EventCode=3
Image!="C:\\Users\\waldo\\AppData\\Local\\Microsoft\\Teams\\current\\Teams.exe"
Image!="C:\\Users\\waldo\\AppData\\Local\\Temp\\*"
Image!="*Microsoft*"
Image!="*OneDrive*"
Image!="*nslookup*"
Image!="*svchost.exe*"
Image!="*Update*"
| stats count by DestinationIp, Image
| sort - count
```

**C2 server connecting in (EventCode 3 by source IP)**

```spl
index="main" SourceIp=10.0.0.186
```

**PsExec password discovery**

```spl
index="main" sourcetype="WinEventLog:Sysmon" password CommandLine="*psexec*"
```

---

### 7.15 TTP-Driven Detection Queries

**Reconnaissance via native Windows binaries (EventCode 1)**

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
Image=*\\ipconfig.exe OR Image=*\\net.exe OR Image=*\\whoami.exe OR Image=*\\netstat.exe
OR Image=*\\nbtstat.exe OR Image=*\\hostname.exe OR Image=*\\tasklist.exe
| stats count by Image,CommandLine
| sort - count
```

**Payloads hosted on githubusercontent.com (EventCode 22)**

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=22 QueryName="*github*"
| stats count by Image, QueryName
```

**PsExec detection via registry (EventCode 13)**

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=13
Image="C:\\Windows\\system32\\services.exe"
TargetObject="HKLM\\System\\CurrentControlSet\\Services\\*\\ImagePath"
| rex field=Details "(?<reg_file_name>[^\\\]+)$"
| eval reg_file_name = lower(reg_file_name), file_name = if(isnull(file_name),reg_file_name,lower(file_name))
| stats values(Image) AS Image, values(Details) AS RegistryDetails, values(_time) AS EventTimes, count by file_name, ComputerName
```

**PsExec detection via file creation (EventCode 11)**

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=11 Image=System
| stats count by TargetFilename
```

**PsExec detection via named pipe (EventCode 18)**

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=18 Image=System
| stats count by PipeName
```

**Archive files used for data exfiltration (EventCode 11)**

```spl
index="main" EventCode=11 (TargetFilename="*.zip" OR TargetFilename="*.rar" OR TargetFilename="*.7z")
| stats count by ComputerName, User, TargetFilename
| sort - count
```

**PowerShell downloading payloads (EventCode 11)**

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=11 Image="*powershell.exe*"
| stats count by Image, TargetFilename
| sort + count
```

**Execution from Downloads folder (EventCode 1)**

```spl
index="main" EventCode=1
| regex Image="C:\\\\Users\\\\.*\\\\Downloads\\\\.*"
| stats count by Image
```

**Executables/DLLs created outside Windows directory (EventCode 11)**

```spl
index="main" EventCode=11 (TargetFilename="*.exe" OR TargetFilename="*.dll")
TargetFilename!="*\\windows\\*"
| stats count by User, TargetFilename
| sort + count
```

**Misspelled PsExec binary detection (EventCode 1)**

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
(CommandLine="*psexe*.exe" NOT (CommandLine="*PSEXESVC.exe" OR CommandLine="*PsExec64.exe"))
OR (ParentCommandLine="*psexe*.exe" NOT (ParentCommandLine="*PSEXESVC.exe" OR ParentCommandLine="*PsExec64.exe"))
OR (ParentImage="*psexe*.exe" NOT (ParentImage="*PSEXESVC.exe" OR ParentImage="*PsExec64.exe"))
OR (Image="*psexe*.exe" NOT (Image="*PSEXESVC.exe" OR Image="*PsExec64.exe"))
| table Image, CommandLine, ParentImage, ParentCommandLine
```

**Non-standard ports (EventCode 3)**

```spl
index="main" EventCode=3
NOT (DestinationPort=80 OR DestinationPort=443 OR DestinationPort=22 OR DestinationPort=21)
| stats count by SourceIp, DestinationIp, DestinationPort
| sort - count
```

---

### 7.16 Analytics-Driven Detection Queries

**Anomalous network connections per process (EventCode 3)**

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=3
| bin _time span=1h
| stats count as NetworkConnections by _time, Image
| streamstats time_window=24h avg(NetworkConnections) as avg stdev(NetworkConnections) as stdev by Image
| eval isOutlier=if(NetworkConnections > (avg + (0.5*stdev)), 1, 0)
| search isOutlier=1
```

**Abnormally long cmd.exe commands (EventCode 1)**

```spl
index="main" sourcetype="WinEventLog:Sysmon" Image=*cmd.exe
| eval len=len(CommandLine)
| table User, len, CommandLine
| sort - len
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" Image=*cmd.exe
ParentImage!="*msiexec.exe" ParentImage!="*explorer.exe"
| eval len=len(CommandLine)
| table User, len, CommandLine
| sort - len
```

**Abnormal cmd.exe execution frequency (EventCode 1)**

```spl
index="main" EventCode=1 (CommandLine="*cmd.exe*")
| bucket _time span=1h
| stats count as cmdCount by _time User CommandLine
| eventstats avg(cmdCount) as avg stdev(cmdCount) as stdev
| eval isOutlier=if(cmdCount > avg+1.5*stdev, 1, 0)
| search isOutlier=1
```

**Processes loading many DLLs quickly (EventCode 7)**

```spl
index="main" EventCode=7
| bucket _time span=1h
| stats dc(ImageLoaded) as unique_dlls_loaded by _time, Image
| where unique_dlls_loaded > 3
| stats count by Image, unique_dlls_loaded
```

```spl
index="main" EventCode=7
NOT (Image="C:\\Windows\\System32*")
NOT (Image="C:\\Program Files (x86)*")
NOT (Image="C:\\Program Files*")
NOT (Image="C:\\ProgramData*")
NOT (Image="C:\\Users\\waldo\\AppData*")
| bucket _time span=1h
| stats dc(ImageLoaded) as unique_dlls_loaded by _time, Image
| where unique_dlls_loaded > 3
| stats count by Image, unique_dlls_loaded
| sort - unique_dlls_loaded
```

**Same process created more than once on same machine (EventCode 1)**

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| transaction ComputerName, Image
| where mvcount(ProcessGuid) > 1
| stats count by Image, ParentImage
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| transaction ComputerName, Image
| where mvcount(ProcessGuid) > 1
| search Image="C:\\Windows\\System32\\rundll32.exe" ParentImage="C:\\Windows\\System32\\svchost.exe"
| table CommandLine, ParentCommandLine
```

---

### 7.17 Sysmon App Exercises

**Net view fix**

```spl
`sysmon` Image="*net.exe" CommandLine="*net  view*"
| stats count by CommandLine
```

**SharpHound network connections fix (EventCode 3)**

```spl
`sysmon` EventCode=3 Image="*SharpHound.exe"
| eval Destination=coalesce(dest_host,dest_ip)
| stats count, values(Destination) AS "Destinations", values(dest_port) AS "Ports", values(protocol) AS "Protocols" by Image
| fields Image Destinations Ports Protocols count
```

---

## 8. Key Sysmon Event IDs Reference

[[Understanding Log Sources & Investigating with Splunk#Embracing The Mindset Of Analysts, Threat Hunters, & Detection Engineers]]

|EventCode|Description|Threat Use Case|
|---|---|---|
|1|Process Creation|Unusual parent-child trees, recon binaries|
|2|File creation time changed|Timestomping|
|3|Network connection|C2, lateral movement|
|5|Process terminated|Cobalt Strike sacrificial processes|
|6|Driver loaded|BYOD driver attacks|
|7|Image (DLL) loaded|DLL hijacking, CLR/C# injection|
|8|CreateRemoteThread|Thread injection|
|10|ProcessAccess|LSASS dumping, memory injection|
|11|FileCreate|File drops, payload staging|
|12/13|RegistryEvent|Persistence, PsExec|
|17/18|Pipe created/connected|PsExec, lateral movement|
|22|DNSEvent|C2 beaconing, payload hosting|
|23|FileDelete|Cleanup, ransomware|
|25|ProcessTampering|Process hollowing/herpadering|

---

## 9. Key Windows Security Event IDs Reference

|EventCode|Description|
|---|---|
|4624|Logon success|
|4662|AD object accessed (DCSync)|
|4768|Kerberos TGT request|
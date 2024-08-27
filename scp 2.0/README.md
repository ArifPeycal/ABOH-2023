# scp 2.0
## Solution
The challenge contains two files, ```Procedure-15A.pdf``` and ```memdump.zip```. 

## Analyzing Procedure-15A.pdf
The PDF contains 2 pages which related to SCP (Secure, Contain, Protect). The 2nd page of the PDF contain  instruction to eradicate evidence from all devices relates to data wiping or secure deletion techniques.
> “All personnel must back up site data including documentation, images, videos and other relevant information to Foundation servers. All evidence and data must then be eradicated from all devices. Please refer to the Lockdown Procedures document for the full documentation for Containment Breaches which follows the lockdown procedure for Site-18. This document is a guidance to all Foundation sites to ensure consistency and that we, as the SCP Foundation, follow our motto. Secure, Contain and Protect.”

 ## Analyzing memdump.zip
 Upon unziping ```memdump.zip```, it shows that there is another file called ```memdump.raw``` inside it. 
> ```.raw``` file is a digital file that contains unprocessed data and usually refers to a forensic image file that represents an exact, bit-by-bit copy of a storage medium (like a hard drive, memory card, or USB stick).

## Memory Analysis using Volatility
I will be using ```Volatility 2.6``` for analysing the image file. First of all, ```imageinfo``` plugin helps identify the profile (OS version and service pack) that matches the memory image. This is crucial because the correct profile needs to be used for further analysis.

```bash
vol.py -f <path/to/file> imageinfo
```
```
Volatility Foundation Volatility Framework 2.6.1
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win7SP1x64, Win7SP0x64, Win2008R2SP0x64, Win2008R2SP1x64_24000, Win2008R2SP1x64_23418, Win2008R2SP1x64, Win7SP1x64_24000, Win7SP1x64_23418
                     AS Layer1 : WindowsAMD64PagedMemory (Kernel AS)
                     AS Layer2 : FileAddressSpace 
                      PAE type : No PAE
                           DTB : 0x187000L
                          KDBG : 0xf80002bed120L
          Number of Processors : 2
     Image Type (Service Pack) : 1
                KPCR for CPU 0 : 0xfffff80002bef000L
                KPCR for CPU 1 : 0xfffff88002f00000L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2023-09-26 05:12:01 UTC+0000
     Image local date and time : 2023-09-26 13:12:01 +0800
```
  Next, I use ```pstree``` plugin to displays a hierarchical view of running processes, similar to the ps command in Unix/Linux or Task Manager in Windows.
```bash
vol.py -f <path/to/file> --profile=Win7SP1x64 pstree
```
```
Name                                                  Pid   PPid   Thds   Hnds Time
-------------------------------------------------- ------ ------ ------ ------ ----
 0xfffffa8007fe25f0:explorer.exe                     2608   2568     49   1258 2023-09-26 05:09:26 UTC+0000
. 0xfffffa800826cb00:vmtoolsd.exe                    2696   2608      9    182 2023-09-26 05:09:27 UTC+0000
. 0xfffffa8006d895f0:MRCv120.exe                     2992   2608     16    312 2023-09-26 05:11:23 UTC+0000
. 0xfffffa8007fb1b00:WINWORD.EXE                     3532   2608     10    426 2023-09-26 05:10:10 UTC+0000
.. 0xfffffa8007469b00:splwow64.exe                   3836   3532      8     73 2023-09-26 05:11:01 UTC+0000
 0xfffffa8008b3b340:winlogon.exe                      552    464      6    118 2023-09-26 05:09:18 UTC+0000
 0xfffffa8008a6f320:csrss.exe                         484    464     10    282 2023-09-26 05:09:18 UTC+0000
 0xfffffa8008a8e9a0:iusb3mon.exe                     2820   2704      5     99 2023-09-26 05:09:27 UTC+0000
```
Upon inspection, ```WINWORD.EXE``` seems interesting for further analysis. 
> ```WINWORD.EXE``` is the executable for Microsoft Word, part of the Microsoft Office suite.

Since the PDF mention about several files, we can analyze files inside the memory. The ```filescan``` plugin scans memory for file objects, which can include open files, files in memory cache, and files that were recently accessed. Send the output to ```.txt``` file for better visibility.
```
vol.py -f <path/to/file> --profile=Win7SP1x64 filescan > output.txt
```
There is a file called ```SCP-055.doc```, probably related to ```WINWORD.EXE``` process. But, it appears to be a mix of various text fragments, file headers, and commands that might have been extracted from a memory dump. This is definitely a dead end.

```mftparser``` is also a good plugin to learn more about files in the system. 
> MFT is a database that stores metadata about every file on an NTFS file system volume. It contains records describing each file’s attributes, such as its name, size, timestamps, permissions, and more.

Even after a file is deleted, its MFT entry and data can remain, making it possible to recover deleted files and investigate past activities on a system. So, we can use ```mftparser``` to see the MFT records.
```bash
vol.py -f <path/to/file> --profile=Win7SP1x64 mftparser > mft.txt
```
Since the flag might already been deleted, we can search for Recycle Bin directory. We found two text files in the Recycle Bin, one contains an encrypted text (base64) and another is path to ```flag.txt```.

```
$FILE_NAME
Creation                       Modified                       MFT Altered                    Access Date                    Name/Path
------------------------------ ------------------------------ ------------------------------ ------------------------------ ---------
2023-09-26 05:10:50 UTC+0000 2023-09-26 05:10:50 UTC+0000   2023-09-26 05:11:12 UTC+0000   2023-09-26 05:10:50 UTC+0000   $Recycle.Bin\S-1-5-~1\$RD0BID3.txt

$OBJECT_ID
Object ID: 46a152d5-2a5c-ee11-8547-c8cb9eccfa14
Birth Volume ID: 80000000-5000-0000-0000-180000000100
Birth Object ID: 31000000-1800-0000-5155-4a505344497a
Birth Domain ID: 65304d77-546c-5130-4d57-35744d303530

$DATA
0000000000: 51 55 4a 50 53 44 49 7a 65 30 4d 77 54 6c 51 30   QUJPSDIze0MwTlQ0
0000000010: 4d 57 35 74 4d 30 35 30 58 30 4a 79 5a 57 46 6a   MW5tM050X0JyZWFj
0000000020: 61 46 38 34 57 56 39 4e 51 47 4e 79 4d 43 52 39   aF84WV9NQGNyMCR9
0000000030: 0a  

$FILE_NAME
Creation                       Modified                       MFT Altered                    Access Date                    Name/Path
------------------------------ ------------------------------ ------------------------------ ------------------------------ ---------
2023-09-26 05:11:18 UTC+0000 2023-09-26 05:11:18 UTC+0000   2023-09-26 05:11:18 UTC+0000   2023-09-26 05:11:18 UTC+0000   $Recycle.Bin\S-1-5-~1\$ID0BID3.txt

$DATA
0000000000: 01 00 00 00 00 00 00 00 31 00 00 00 00 00 00 00   ........1.......
0000000010: 30 ef 87 e1 37 f0 d9 01 43 00 3a 00 5c 00 55 00   0...7...C.:.\.U.
0000000020: 73 00 65 00 72 00 73 00 5c 00 61 00 64 00 6d 00   s.e.r.s.\.a.d.m.
0000000030: 69 00 6e 00 5c 00 44 00 65 00 73 00 6b 00 74 00   i.n.\.D.e.s.k.t.
0000000040: 6f 00 70 00 5c 00 66 00 6c 00 61 00 67 00 2e 00   o.p.\.f.l.a.g...
0000000050: 74 00 78 00 74 00 00 00 00 00 00 00 00 00 00 00   t.x.t...........
```
By using Powershell command, we can decrypt the base64 text and get the flag.
```powershell
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String('QUJPSDIze0MwTlQ0MW5tM050X0JyZWFjaF84WV9NQGNyMCR9'))
```

## Flag
```ABOH23{C0NT41nm3Nt_Breach_8Y_M@cr0$}```

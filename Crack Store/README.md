# Crack Store
## Challenge Overview
The challenge file is a ZIP archive named ```backup.zip``` using the "Store" compression method. When attempting to unzip, it prompts for a password. Even with incorrect passwords, the ZIP file reveals the filenames within, which resemble those typically found in a Windows System's C:\Windows\System32\drivers\etc directory. <br>

![image](https://github.com/user-attachments/assets/0c82c486-1bc1-40ea-9347-b6d0d86bee7c)
## Initial Thoughts and Approach
At first, I assumed the challenge involved a dictionary attack using zip2john and john, based on the challenge's name. However, after failing and asking the creator for a hint, I realized the challenge's name was the hint itself. A quick Google search on "ZIP Store Crack" led me to <a href="https://www.acceis.fr/cracking-encrypted-archives-pkzip-zip-zipcrypto-winzip-zip-aes-7-zip-rar/">an article</a> about cracking encrypted archives using the Biham and Kocher plaintext attack, which is effective against ZIP files using the ZipCrypto Store compression method.

## Solution 
The article specified that the Biham and Kocher attack requires at least 12 bytes of known plaintext, with 8 bytes needing to be contiguous. Observing the file names and contents, I found that files like networks and protocols on my system had consistent starting bytes, such as ```# Copyright (c) 1993-1999 Microsoft Corp```, providing a good amount of known plaintext. <br>

In addition, the XML files often start with ```<?xml version="1.0"?>```, making them suitable for this attack.

1. Extract Known Plaintext:

   - I used the networks file, which had a known header of 40 bytes.
2. Run bkcrack to Identify Encryption Keys:
   - This command identifies possible encryption keys using the known plaintext.

```bash
bkcrack -C backup.zip -c networks -p networks
```

![image](https://github.com/user-attachments/assets/d0c2f607-8dd8-4437-80ab-bd1c6eb08d45)

3. Find the Password:
    - Here, <encryption keys> are the keys identified in the previous step, and \?a specifies that the password can include any ASCII character.
```bash
bkcrack -k <encryption keys> -r <Max estimated length of password> \?a
```
![image](https://github.com/user-attachments/assets/3237c930-f87e-4325-984d-2135740041cd)

The password turned out to be ```JOhnH@mM0nD```, which successfully extracted the files from the ZIP archive. 

## Flag 
```
ABOH23{n0t_That_KINd_oF_CR4Ck_S70RE}
```

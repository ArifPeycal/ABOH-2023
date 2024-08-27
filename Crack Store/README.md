# Crack Store
## Solution
The challenge file is a ZIP archive named ```backup.zip``` using the "Store" compression method. When attempting to unzip, it prompts for a password. Even with incorrect passwords, the ZIP file reveals the filenames within, which resemble those typically found in a Windows System's C:\Windows\System32\drivers\etc directory.
![image](https://github.com/user-attachments/assets/0c82c486-1bc1-40ea-9347-b6d0d86bee7c)

At first, I assumed the challenge involved a dictionary attack using zip2john and john, based on the challenge's name. However, after failing and asking the creator for a hint, I realized the challenge's name was the hint itself. A quick Google search on "ZIP Store Crack" led me to <a href="https://www.acceis.fr/cracking-encrypted-archives-pkzip-zip-zipcrypto-winzip-zip-aes-7-zip-rar/">an article</a> about cracking encrypted archives using the Biham and Kocher plaintext attack, which is effective against ZIP files using the ZipCrypto Store compression method.

It is stated in the article that to conduct this attack, it requires at least 12 bytes of known plaintext and at least 8 of them must be contiguous. 

What I noticed is that the same files in my host machine contain similar strings at the start of the file.  For example, ```networks``` has ```# Copyright (c) 1993-1999 Microsoft Corp``` and ```protocols``` has ```# Copyright (c) 1993-2006 Microsoft Corp```. This is sufficient for our attack (40 bytes long). The article also stated that ```svg``` also can be a good candidate for the attack since most of it starts with ```<?xml version="1.0"?```. 

Using bkcrack, I executed the following commands:

- To identify the encryption keys.

```bash
bkcrack -C backup.zip -c networks -p networks
```
![image](https://github.com/user-attachments/assets/d0c2f607-8dd8-4437-80ab-bd1c6eb08d45)

- To find the password, allowing any ASCII character.
  
```bash
bkcrack -k <encryption keys> -r <Max estimated length of password> \?a
```
![image](https://github.com/user-attachments/assets/3237c930-f87e-4325-984d-2135740041cd)

The password turned out to be ```JOhnH@mM0nD```, which successfully extracted the files from the ZIP archive. 

## Flag 
```ABOH23{n0t_That_KINd_oF_CR4Ck_S70RE}```

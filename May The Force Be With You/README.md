# May The Force Be With You
## Challenge Overview
In this challenge, we were given an encrypted file along with the encryption script. The script used AES encryption in CBC mode, with a key derived using PBKDF2. The challenge was to decrypt the encrypted file to retrieve the original plaintext.
```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from Crypto.Random import get_random_bytes
from Crypto.Protocol.KDF import PBKDF2
import textwrap

def encrypt_file(file_path, password):
    with open(file_path, 'rb') as file:
        plaintext = file.read()

    iv = get_random_bytes(AES.block_size)
    passwd = textwrap.dedent(password)[:-1]
    salt = b'salt123'  
    key = PBKDF2(passwd.encode(), salt, dkLen=16)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    ciphertext = cipher.encrypt(pad(plaintext, AES.block_size))
    encrypted_file_path = file_path + '.enc'

    with open(encrypted_file_path, 'wb') as file:
        file.write(ciphertext + iv)
    print("Encryption successful. Encrypted file saved as:", encrypted_file_path)

password = "ni5h2h?Yrq8Do?n+|6a;pKbZkv%}O~tV" 
file_path = "./flag.txt"   
encrypt_file(file_path, password)
```
## Encryption Script Analysis
The provided script encrypts a file using the following steps:

1. Key Derivation: The key is derived using PBKDF2 with a known salt (```salt123```) and the given password (```ni5h2h?Yrq8Do?n+|6a;pKbZkv%}O~tV```). The derived key is 16 bytes long.

2. Initialization Vector (IV): A random IV is generated, which is essential for decrypting the data.

3. AES Encryption: The script uses AES in CBC mode to encrypt the padded plaintext.

4. Output: The ciphertext is saved to a new file with the IV appended at the end.

## Decryption Process
To decrypt the file, we need to reverse the encryption process using the same key derivation method and the appended IV.

Here's the decryption script

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
from Crypto.Protocol.KDF import PBKDF2

def decrypt_file(encrypted_file_path, password):
    with open(encrypted_file_path, 'rb') as file:
        ciphertext = file.read()
    
    # Extract the IV from the last 16 bytes of the ciphertext
    iv = ciphertext[-AES.block_size:]
    ciphertext = ciphertext[:-AES.block_size]

    # Derive the key using the same salt and password
    salt = b'salt123'
    key = PBKDF2(password.encode(), salt, dkLen=16)

    # Create a new AES cipher object with the extracted IV
    cipher = AES.new(key, AES.MODE_CBC, iv)

    # Decrypt the ciphertext and remove padding
    plaintext = unpad(cipher.decrypt(ciphertext), AES.block_size)
    
    # Output the decrypted content
    print("Decrypted content:", plaintext.decode())

# Use the same password that was used for encryption
password = "ni5h2h?Yrq8Do?n+|6a;pKbZkv%}O~tV"
encrypted_file_path = "./flag.txt.enc"
decrypt_file(encrypted_file_path, password)
```
### Steps to Decrypt:
1. Extract the IV: The script extracts the IV, which is appended to the end of the ciphertext.

2. Key Derivation: The script reuses the known salt and password to derive the decryption key.

3. Decrypt the Ciphertext: The ciphertext is decrypted using AES in CBC mode, and the plaintext is unpadded to reveal the original content.

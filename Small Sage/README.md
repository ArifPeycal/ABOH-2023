# Small Sage
## Challenge Overview 
In this challenge, you are given a SageMath script that performs RSA encryption with a small public exponent \( e = 3 \). The script generates two large prime numbers \( p \) and \( q \), calculates the modulus \( n = p \times q \), and encrypts a message that includes the flag.

### Breakdown of the Script

1. **Prime Generation:**
   ```python
   p, q = random_prime(2 ^ 1024), random_prime(2 ^ 1024)
   ```
   Here, two 1024-bit prime numbers \( p \) and \( q \) are generated using `random_prime`.

2. **Modulus Calculation:**
   ```python
   n = p * q
   ```
   The modulus \( n \) is the product of \( p \) and \( q \), which is typical in RSA encryption.

3. **Public Exponent:**
   ```python
   e = 3
   ```
   The public exponent \( e \) is set to 3, a common choice in RSA encryption due to its efficiency.

4. **Message Preparation:**
   ```python
   FLAG = open("flag.txt", "rb").read().strip()
   m = bytes_to_long(FLAG + b' is your challenge flag.')
   ```
   The flag is read from the file `flag.txt` and concatenated with the string `' is your challenge flag.'`. This combined message is then converted into a long integer using `bytes_to_long`.

5. **Encryption:**
   ```python
   c = pow(m, e, n)
   ```
   The message \( m \) is encrypted using the formula \( c = m^e \mod n \), where \( c \) is the ciphertext.

6. **Output:**
   ```python
   print("N: ", n)
   print("C: ", c)
   print("e: ", e)
   ```
   The script outputs the modulus \( n \), the ciphertext \( c \), and the public exponent \( e \).

## Solution

Since the public exponent \( e \) is small, and if the plaintext message \( m \) is small enough (i.e., \( m^e < n \)), the ciphertext \( c \) can be easily decrypted by taking the cube root of \( c \) (since \( e = 3 \)).

```python
import gmpy2

# Given values
n = 28161864534081810305839467239167774824180698442991360538137338315924601027539535041400325106523598882827263670671140966855944057889837783992080270143420119844958855679728614805589197733901663249220100214524859116110365815705699485099116276988534253521580223115836247118089590595980346272692504104976860138248959015932618979651746563030552421216691329694961700647328850519321776696007920491542096366696034760558758393690945535590284240994579352805664119144134863786797266463118165575746650538843159490903440899114347091988968775074879305009340592457617508211781199057573663246634610497629416920053419998682083393087987 # The value of n from the challenge output
c = 762355112596222421309825166446067448121886093544068458795156044255325081286699861240486430215279901835675723822721970949307265398924333599178805487220325668055743991293697494477706560130827449405781098938392283482757063955895656607033694619449376928780098570577226994800731087835230561205556094959240210387000  # The value of c from the challenge output
e = 3

# Find the cube root of the ciphertext
m, exact = gmpy2.iroot(c, e)

if exact:
    # Convert the long integer back to bytes
    m_bytes = m.to_bytes((m.bit_length() + 7) // 8, byteorder='big')
    # Strip the known suffix to get the flag
    flag = m_bytes.rstrip(b' is your challenge flag.')
    print("Flag:", flag.decode())
else:
    print("Cube root is not exact. The plaintext might have been too large.")
```
### Explanation
1. gmpy2.iroot(c, e): This calculates the integer eth root of c (in this case, the cube root).The iroot function returns a tuple with the root and a boolean indicating whether the root was exact.
2. m.to_bytes(...): Convert the integer message back to its original byte form.
3. rstrip: Strip the known suffix to reveal the flag.

## Flag
```
ABOH23{rocky0ubrr!}
```

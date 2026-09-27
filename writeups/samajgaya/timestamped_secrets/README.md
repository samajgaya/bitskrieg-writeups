# Timestamped Secrets - Cryptography (medium)

Attached are two files, a python script and a text file.
The text file contains a Hint, which gives us a timestamp of when the message
was encrypted, and the hex-encoded cyphertext.
The python script seems to be the script used to encode the secret. It uses
the current time as the key for encryption.
Since we have the ciphertext and the required key, decrypting the ciphertext
should be straight forward.

```python
from hashlib import sha256
from Crypto.Cipher import AES

timestamp = 1770242597
ciphertext = bytes.fromhex(input())

key = sha256(str(timestamp).encode()).digest()[:16]
cipher = AES.new(key, AES.MODE_ECB)

cipher.decrypt(ciphertext)
```
and we have the flag!

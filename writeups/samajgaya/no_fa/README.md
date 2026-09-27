# No FA - Web Exploitation (medium)

## Overview
Provided are two files, an `app.py` and `users.db`.
Reading `app.py` makes it clear that we need to login as `admin` to get the
flag, and that the database schema contains columns for `users` and their 
sha256 password hash (unsalted) in `password`.

## Solving
Using this "leaked" databasse to find the `admin` users password hash, and
using hashcat to crack it yields a password quite quickly!

Logging into the web interface with `admin` and the cracked password, brings
us to an OTP verification page, where a 4 digit OTP is prompted.
Checking `app.py` for the specifics shows that the OTP is a random 4 digit
number (as expected), stored in the flask session storage cookie. This is
then compared to what the user inputs, and given that the OTP hasn't expired
and matches the OTP generated, the user logs in successfully.

Flask should default to client side secure storage of cookies, which is
signed but not encrypted. Using the browsers cookie for our session we should
be able to get the cookie and in-turn get the required OTP.

Using Flask's default cookie storage schema:
```
[.]payload.timestamp.signature
```
where payload is base64 encoded and compressed, all we need to extract the OTP
is:

```python
import base64
import zlib

cookie = input()

payload = cookie.lstrip(".").split(".", 1)[0]

payload += "=" * (-len(payload) % 4)
compressed = base64.urlsafe_b64decode(payload)
data = zlib.decompress(compressed)

print(data.decode())
```
This prints out the OTP and using that in the web interface, we successfully
login and have the flag!

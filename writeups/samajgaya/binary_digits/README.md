# Binary Digits - Forensics (easy)

## Overview
Attached is a text file `digits.bin`
```sh
$ file digits.bin
digits.bin: ASCII text, with very long lines (65536), with no line termin ators
```
It contains a textual representation of binary data.
Clearly, it isn't ascii as grepping for '\0' gives more than 1 result.

## Solving
Let's try converting this textual binary to an actual binary file.
```python
data = # 0b... , copy pasted the text representation of binary as a binary number

# 2. Convert the integer into raw bytes
byte_length = (binary.bit_length() + 7) // 8 # minumum number of bytes to store

# big-endian is one-to-one
binary_data = binary.to_bytes(byte_length, byteorder='big')

# write to file
with open('out', 'wb') as f:
    f.write(binary_data)
```

now trying to check file-type again
```sh
$ file out
ofile: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1,
segment length 16, baseline, precision 8, 800x500, components 3
```
an image file!
Opening with an image viewer, gives us the flag!

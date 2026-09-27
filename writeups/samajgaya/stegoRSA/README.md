# StegoRSA - Cryptography (easy)

Attached are two files:
1. an RSA encrypted flag as indicated by the problem description, encoded in
   raw binary.
2. a jpeg image file, which is an image of a key (clever!).

Running exiftool on the jpeg image gives 'interesting' output:
```sh
$ exiftool image.jpg
ExifTool Version Number         : 13.57
File Name                       : image.jpg
Directory                       : .
File Size                       : 21 kB
File Modification Date/Time     : 2026:02:07 20:56:34+05:30
File Access Date/Time           : 2026:09:18 12:44:10+05:30
File Inode Change Date/Time     : 2026:09:18 12:44:08+05:30
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Comment                         : << really long hex-encoding stuff >>
Image Width                     : 512
Image Height                    : 512
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 512x512
Megapixels                      : 0.262
```

The comment field seems to hold some sort of data, likely hex-encoded. Trying
to extract and decode with `xxd(1)` gives:
```sh
$ exiftool -Comment -b image.jpg | xxd -r -p 
-----BEGIN PRIVATE KEY-----

+++ <REDACTED> +++

-----END PRIVATE KEY-----
```

Writing this to a file `private.pem`, and using it to decrypt the flag raw
gives the flag!

```sh
$ openssl pkeyutl -decrypt -inkey private.pem -in flag.enc
picoCTF{...}
```

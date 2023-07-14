---
title: "baby AES"
tags: "Cryptoverse CTF"
---

We are given this simple code, that shows an AES-CBC encryption, moreover it gives us both the value of the IV and the ciphertext. The big problem here is the key, that is only 2 bytes long, so we can bruteforce it. This is the code:

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from secret import flag

KEY_LEN = 2
BS = 16
key = pad(open("/dev/urandom","rb").read(KEY_LEN), BS)
iv =  open("/dev/urandom","rb").read(BS)

cipher = AES.new(key, AES.MODE_CBC, iv)
ct = cipher.encrypt(pad(flag, 16))

print(f"iv = {iv.hex()}")
print(f"ct = {ct.hex()}")

# Output:
# iv = 1df49bc50bc2432bd336b4609f2104f7
# ct = a40c6502436e3a21dd63c1553e4816967a75dfc0c7b90328f00af93f0094ed62
```

We can use the following code to bruteforce the key and retrieve the flag:

```python
import itertools
from Crypto.Util.Padding import pad
from Crypto.Cipher import AES


KEY_LEN = 2
BS = 16
iv = bytes.fromhex('1df49bc50bc2432bd336b4609f2104f7')
ct = bytes.fromhex('a40c6502436e3a21dd63c1553e4816967a75dfc0c7b90328f00af93f0094ed62')

def bruteforce():
    for i in range(0xff):
        b0 = bytes([i])
        for j in range(0xff):
            b1 = bytes([j])
            key = pad(b0+b1, BS)
            cipher = AES.new(key, AES.MODE_CBC, iv)
            try:
                flag = cipher.decrypt(ct)
                if b'cvctf{' in flag:
                    print(flag)
            except:
                continue


bruteforce()

# cvctf{b4by_AES_s1mpL3}
```
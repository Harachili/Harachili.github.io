---
title: "lazy platform AES"
tags: "Team Italy CTF 2022"
---

This challenge needs a connection with a server, on which runs an oracle. This oracle encrypts, and gives us the ciphertext, the key and the iv (unless it encrypts the flag, for which only gives the ct). This is the code of the oracle:  

```python
from Crypto.Util.Padding import pad
from Crypto.Cipher import AES
import random
import signal
import os

TIMEOUT = 300
FLAG = os.environ.get("FLAG", "flag{test}").encode()


def getrandbytes(n: int) -> bytes:
    return random.getrandbits(n * 8).to_bytes(n, "little")


def handle():
    print("Welcome to Lazy platform! If you want to decrypt some messages, you can't do that here, you'll have to do it on your own")

    while True:
        print("Choose one of the following options")
        print("[1] Encrypt")
        print("[2] Decrypt")
        print("[3] Get encrypted flag")
        print("[4] Exit")
        option = input("> ")

        if option == "1":
            message = input("Enter a message to encrypt: ")
            key = getrandbytes(32)
            iv = getrandbytes(16)
            ciphertext = AES.new(key, AES.MODE_CBC, iv).encrypt(
                pad(message.encode(), AES.block_size))
            print("Ciphertext:", ciphertext.hex())
            print("Key:", key.hex())
            print("IV:", iv.hex())
        elif option == "2":
            print("I can't do that at the moment, I'm cooking a pizza")
        elif option == "3":
            key = getrandbytes(32)
            iv = getrandbytes(16)
            ciphertext = AES.new(key, AES.MODE_CBC, iv).encrypt(
                pad(FLAG, AES.block_size))
            print("Ciphertext:", ciphertext.hex())
        elif option == "4":
            print("Bye bye!\n")
            break
        else:
            print("Invalid option")
        print()


if __name__ == "__main__":
    signal.alarm(TIMEOUT)
    handle()
```

To solve this challenge, it's enough to send requests to the server up to the point in which we break the RNG of the random library, using [RandCrack](https://github.com/tna0y/Python-random-module-cracker).  
This is the implemented code to find the flag:  

```python
#! /usr/bin/python3

from pwn import *
from randcrack import RandCrack
from struct import unpack
from binascii import unhexlify
import codecs

def getrandbytes(n: int) -> bytes:
    return rc.predict_getrandbits(n * 8).to_bytes(n, "little")

r = remote('lazy-platform.challs.teamitaly.eu', 15004 )

rc = RandCrack()

while not rc.state:
    r.sendlineafter(b'> ', b'1')
    r.sendlineafter(b': ', b'')
    r.readline()
    key1 = r.readlineS().strip().split(' ')[1]
    iv1  = r.readlineS().strip().split(' ')[1]
    
    tmp = int.from_bytes(codecs.decode(key1,'hex_codec'), 'little')
    while tmp>0:
        print("key")
        rc.submit(tmp % (1 << 32))
        tmp = tmp>>32
    
    tmp = int.from_bytes(codecs.decode(iv1,'hex_codec'), 'little')
    while tmp>0:
        print("iv")
        rc.submit(tmp % (1 << 32))
        tmp = tmp>>32


keypr  = getrandbytes(32)
ivpred = getrandbytes(16)

print(keypr.hex())
print(ivpred.hex())

r.sendlineafter(b'> ', b'3')
r.interactive()

# # flag{u53_s3cure_r4nd0m_numb3r_gen3r4t0rs}
```

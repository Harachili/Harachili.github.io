---
title: "aes-cfb with leaks"
tags: "HTB-2023"
---

We are given a source code which is the following:

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
import random, os

FLAG = b'HTB{??????????????????????}'


class CAES:

    def __init__(self):
        self.key = os.urandom(16)
        self.cipher = AES.new(self.key, AES.MODE_ECB)

    def blockify(self, message, size):
        return [message[i:i + size] for i in range(0, len(message), size)]

    def xor(self, a, b):
        return b''.join([bytes([_a ^ _b]) for _a, _b in zip(a, b)])

    def encrypt(self, message):
        iv = os.urandom(16)

        ciphertext = b''
        plaintext = iv

        blocks = self.blockify(message, 16)
        for block in blocks:
            ct = self.cipher.encrypt(plaintext)
            encrypted_block = self.xor(block, ct)
            ciphertext += encrypted_block
            plaintext = encrypted_block

        return ciphertext

    def leak(self, blocks):
        r = random.randint(0, len(blocks) - 2)
        leak = [self.cipher.encrypt(blocks[i]).hex() for i in [r, r + 1]]
        return r, leak


def main():
    aes = CAES()
    message = pad(FLAG * 4, 16)

    ciphertext = aes.encrypt(message)
    ciphertext_blocks = aes.blockify(ciphertext, 16)

    r, leak = aes.leak(ciphertext_blocks)

    with open('output.txt', 'w') as f:
        f.write(f'ct = {ciphertext.hex()}\nr = {r}\nphrases = {leak}\n')


if __name__ == "__main__":
    main()
```

Analyzing it we can notice that we have a random key, a random IV and a random number r. The key is used to encrypt the IV with AES-ECB and then the IV is used to encrypt the plaintext with AES-CFB. The ciphertext is then divided in blocks of 16 bytes and the block at index r and r+1 are encrypted with AES-ECB and then converted to hex. The ciphertext, r and the two encrypted blocks are then written in a file.  
Using the leaked information we can really simply recover the plaintext, since we know that AES-CFB is really weak if the same IV is used to encrypt two different plaintexts.  
In fact, we can simply xor the two encrypted blocks with the two leaked blocks to retrieve the plaintext:
    
```python
from pwn import *

def blockify(message, size):
    return [message[i:i + size] for i in range(0, len(message), size)]

ct = bytes.fromhex('bc9bc77a809b7f618522d36ef7765e1cad359eef39f0eaa5dc5d85f3ab249e788c9bc36e11d72eee281d1a645027bd96a363c0e24efc6b5caa552b2df4979a5ad41e405576d415a5272ba730e27c593eb2c725031a52b7aa92df4c4e26f116c631630b5d23f11775804a688e5e4d5624')
ct_blocks = blockify(ct, 16)

leak_3 = bytes.fromhex('8b6973611d8b62941043f85cd1483244')
leak_4 = bytes.fromhex('cf8f71416111f1e8cdee791151c222ad')
print(xor(ct_blocks[4], leak_3))
print(xor(ct_blocks[5], leak_4))
# HTB{CFB_15_w34k_w17h_l34kz}
```
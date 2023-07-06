---
title: "The oracle's apprentice"
tags: "HeroCTF v4"
---

In this challenge we are given this source code:
    
```python
#!/usr/bin/env python3
from Crypto.Util.number import getStrongPrime, bytes_to_long
import random

FLAG = open('flag.txt','rb').read()

encrypt = lambda m: pow(m, e, n)
decrypt = lambda c: pow(c, d, n)

e = random.randrange(3, 65537, 2) # odd number between 3 and 65537
p = getStrongPrime(1024, e=e) # prime such that p - 1 is coprime to e
q = getStrongPrime(1024, e=e) # prime such that q - 1 is coprime to e

n = p * q
φ = (p-1) * (q-1) 

d = pow(e, -1, φ)

c = encrypt(bytes_to_long(FLAG))

#print(f"{n=}")
#print(f"{e=}")
print(f"{c=}")

for _ in range(3):
     t = int(input("c="))
     print(decrypt(t)) if c != t else None
```

As we can see, we have a simple oracle that decrypts whatever we give them, except if it is the flag.  
To solve this problem, is enough to know that RSA is a homomorphic encryption. From this we can notice that:  
`c*c = m^(2*e) mod n`  
So, once it get decrypted we have:  
`m^(2*e)^d mod n = m^(2*e*d) mod n = m^2 mod n`  
So, we can simply take the square root of the ciphertext and we will get the flag.

Final script:
```python
from pwn import *
from Crypto.Util.number import long_to_bytes as ltb
from gmpy2 import iroot

r = remote("crypto.heroctf.fr", 9000)

r.recvuntil(b"c=")
c = int(r.recvline().strip())

print("cipher = ", c)

attempt = c*c # I calculate m**(2*e) (mod n)

#r.sendlineafter(b"c=", b"2")
r.sendlineafter(b"c=", str(attempt).encode())

p1 = int(r.recvline().strip())


print(ltb(iroot(p1, 2)[0]))

# Hero{m4ybe_le4ving_the_1nt3rn_run_th3_plac3_wasnt_a_g00d_id3a}

""" Intended solution, as described by the author:
TL;DR : send -1, as dec(-1) <=> n - 1
send 2 and recover e (using 2 and dec(2)
send c * enc(2), as dec(c * enc(2)) <=> m * 2 and divide to get m
"""
```

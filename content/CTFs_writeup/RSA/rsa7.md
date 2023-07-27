---
title: "too-many-keys"
tags: "OliCyberIT"
---
<br></br>
We begin by reading the description of the challenge:  
```txt
Da un furto siamo riusciti a recuperare tutte queste chiavi private. Riusciresti a decrittare il messaggio? Sembra che queste chiavi siano state generate tutte allo stesso istante.
```
Which translated to english is:
```txt
From a theft we managed to recover all these private keys. Could you decrypt the message? It seems that these keys were all generated at the same time.
```

We are given a `.zip` file containing 50 RSA public keys, a `flag.txt.enc` file and another public key called `root.pem`.  

The `.pem` files looked like this (in particular this is `root.pem`), which is a public key in PEM format:  
```txt
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA1SvStJBdXpnNf4QKf5Lb
u8c7wZvVrsxE9XsqNpCZSnI+qoiz8LO4PSOeYm1kyPAzm45teUO7kQQ8f+Bl+oPQ
wJWYwRT1HSkn5m8MbUt1NDO0hfDUeak5UHLv4HtxMYEjntaX2pwDBc7+R5EP/vHS
rADYF6QJPlk3UJhR/91rZn8aA6HYLzY+aveK7ogmUVEfPBgpse25WpWa0DkCkgWV
mWq+T+Za24lnMtkguv41xBA9AXE/67l2DnAIxUPnqpCX+NKERBvB90mvEqlCwDBV
tQ9y6igvMrqyTI1WxRmat/pDqBnBGrkh9ROZ5h8PJ0wUgrXKA3sZ1l0fITyj6f3z
1QIDAQAB
-----END PUBLIC KEY-----
```

I guessed that there might have been two public keys with the same modulus, so I tried to go towards that path; I wrote a script to check if there were any public keys with the same modulus, and I found out that the `root.pem` and `key_42.pem` (n1c3) had a common modulus, win!  
Once I found this, the rest was straightforward: I just had to calculate the private key and decrypt the flag, this is the final script I used:
```python
from Crypto.PublicKey import RSA
from math import gcd
from Crypto.Cipher import PKCS1_v1_5
from Crypto.Random import get_random_bytes

l = []
ns = []
es = []

f = open('root.pem','r')
key = RSA.importKey(f.read())
ns.append(key.n)
e = key.e

for i in range(1, 51): 
	fileString = 'keys/key_' + str(i) + '.pem'
	f = open(fileString,'r')
	key = RSA.importKey(f.read())
	ns.append(key.n)

 
lung = len(ns)

p = 0

for i in range(lung):
	for j in range(lung):
		if i != j:
			mcd = gcd(ns[i], ns[j])
			if mcd != 1:
				print(f"Sium! \nFirst index: {i}\nSecond index: {j}\nCommon factor: {mcd}\n")
				p = mcd
				n = ns[i]
				break
	if p != 0:
		break



q = n // p

phiN = (p-1)*(q-1)

d = pow(e, -1, phiN)

newkey = RSA.construct((n, e, d, p, q))
with open("flag.txt.enc", mode='rb') as file:
    fileContent = file.read()
cipher_rsa = PKCS1_v1_5.new(newkey)
print(cipher_rsa.decrypt(fileContent, get_random_bytes(16)))

# flag{t00_many_k3ys_me4ns_a_l0t_of_primes}
```
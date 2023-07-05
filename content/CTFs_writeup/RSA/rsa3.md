We are given a source code:
```python
#!/usr/bin/env python

from Crypto.Util.number import *
from secret import exp, flag, nbit

assert exp & (exp + 1) == 0 

def adlit(x):
    l = len(bin(x)[2:])
    return (2 ** l - 1) ^ x 

def genadlit(nbit):
    while True:
        p = getPrime(nbit)
        q = adlit(p) + 31337
        if isPrime(q):
            return p, q

p, q = genadlit(nbit)
e, n = exp, p * q

c = pow(bytes_to_long(flag), e, n)

print 'n =', hex(n)
print 'c =', hex(c)
```
and the output of the program:
```
n = 0x3ff77ad8783e006b6a2c9857f2f13a9d896297558e7c986c491e30c1a920512a0bad9f07c5569cf998fc35a3071de9d8b0f5ada4f8767b828e35044abce5dcf88f80d1c0a0b682605cce776a184e1bcb8118790fff92dc519d24f998a9c04faf43c434bef6c0fa39a3db7452dc07ccfced9271799f37d91d56b5f21c51651d6a9a41ee5a8af17a2f945fac2b1a0ea98bc70ef0f3e37371c9c7b6f90d3d811212fc80e0abcd5bbefe0c6edb3ca6845ded90677ccd8ff4de2c747b37265fc1250ba9aa89b4fd2bdfb4b4b72a7ff5b5ee67e81fd25027b6cb49db610ec60a05016e125ce0848f2c32bff33eed415a6d227262b338b0d1f3803d83977341c0d3638f
c = 0x2672cade2272f3024fd2d1984ea1b8e54809977e7a8c70a07e2560f39e6fcce0e292426e28df51492dec67d000d640f3e5b4c6c447845e70d1432a3c816a33da6a276b0baabd0111279c9f267a90333625425b1d73f1cdc254ded2ad54955914824fc99e65b3dea3e365cfb1dce6e025986b2485b6c13ca0ee73c2433cf0ca0265afe42cbf647b5c721a6e51514220bab8fcb9cff570a6922bceb12e9d61115357afe1705bda3c3f0b647ba37711c560b75841135198cc076d0a52c74f9802760c1f881887cc3e50b7e0ff36f0d9fa1bfc66dff717f032c066b555e315cb07e3df13774eaa70b18ea1bb3ea0fd1227d4bac84be2660552d3885c79815baef661
```

As I start analyzing the code, I notice that, because of the assertion, the e must be in the form 2**k - 1 for some k.  
Then I see that the function adlit(x) returns the bitwise xor of x with a number with the same number of bits of x, all set to 1.  
  
I calculate p + q = p + (adlit(p) + 31337) = (p + adlit(p)) + 31337 = (2\*\*k - 1) + 31337 = 2\*\*k + 31336  
=> we have that p + q = 2\*\*k + 31336 and that n = pq  
R = 2\*\*k + 31336  
q = R - p  
n = p\*(R - p)  
p\*\*2 + R\*p - n = 0  
=> I solve the second degree equation and find p and q

```python
import gmpy2 
from Crypto.Util.number import long_to_bytes
n = ...
c = ...

def get_pq(k):  		# find p and q knowing p is the bitflip of q
    R = 2**k + 31336

    temp = R**2 - 4*n
    if temp < 0: return -1, -1
    sq, flag = gmpy2.iroot(temp, 2)
    if not flag: return -1, -1
    p = (R + sq) // 2
    q = (R - sq) // 2
    return p, q

k = len(bin(n)[2:]) // 2 + 1
p, q = get_pq(k) 
phi = (p-1)*(q-1) 

print('nbit =', k)
print('p    =', p)
print('q    =', q)


```

At this point I try to brute-force and looking for values with a gcd not too large compared to phiN.
```python
k = 1
while True:				# find e (bruteforce)
    k += 1
    e = 2**k - 1

    gcd =  gmpy2.gcd(e, phi)
    
    # Ignore if gcd is too high
    if gcd > 8: continue
    if e % gcd**2 == 0: continue
    e_prime = e // gcd

    d_prime = gmpy2.invert(e_prime, phi)
    
    # RSA decryption
    p_g = pow(c, d_prime, n)
    
    p, is_valid = gmpy2.iroot(p_g, gcd)
    plaintext = long_to_bytes(p)
    if b'CCTF' in plaintext and is_valid:
        print(e)
        break

print(plaintext)
```
---
title: "Non-Quadratic-Residue"
tags: "AmateursCTF - 2023"
---

We are given two files: a python code (main.py) and its output (output.txt).
The content of both is the following:

```python
from Crypto.Util.number import *
from flag import flag


def getModPrime(modulus):
    p = getPrime(1024)
    p += (modulus - p) % modulus
    p += 1
    iters = 0
    while not isPrime(p):
        p += modulus
    return p


difficulty = 210

flag = bytes_to_long(flag)

# no cheeses here!
b = getModPrime(difficulty**10)

c = inverse(flag, b)
n = pow(c, difficulty, b)

with open('output.txt', 'w') as f:
    f.write(f"{n} {b}")

```
    
```txt
13350651294793393689107684390908420972977381011191079381202728507002264420264784588373703945341668404762890725356808809021906408198983625375190500172144348596288910240548668158058030780501343680214713780242304547715977777103636873360269427453504233184515002477489763359569764117968027273137245802436961373256 135954424160848276393136392848608760791498666756786983317146989739232222268153235587604168914827859099133726281621143020610041450200631778336472889038077986687446107427527703447531968569919642975653169056203851297117178187249653136191818357235077367060617558261023389453028554177668515375377299577050000000001
```

The first thing that came to my mind was to try to find the flag by retrieving first the value of `c`, since it is noticeable that `c = flag^d [mod b]`, so once we have it, finding the flag is as simple as solving the roots of the polynomial `flag^d - c = 0 [mod b]`.  
Once I noticed this, it was almost straightforward to find the flag, since seeing how n is calculated (`n = pow(c, difficulty, b)`), we can find the value of `c` by calculating the multiplicative inverse of `n [mod b]`.  
This is the code I used to find the flag:

```python
from Crypto.Util.number import *

# b = k*d**10 + 1
# c = n^-1 [mod b]
# c = flag^d [mod b]
# => in (flag^d - c).roots() there is the flag -> win!

n, b = 13350651294793393689107684390908420972977381011191079381202728507002264420264784588373703945341668404762890725356808809021906408198983625375190500172144348596288910240548668158058030780501343680214713780242304547715977777103636873360269427453504233184515002477489763359569764117968027273137245802436961373256, 135954424160848276393136392848608760791498666756786983317146989739232222268153235587604168914827859099133726281621143020610041450200631778336472889038077986687446107427527703447531968569919642975653169056203851297117178187249653136191818357235077367060617558261023389453028554177668515375377299577050000000001
d = 210 # difficulty

c = pow(n, -1, b)


Px = PolynomialRing(GF(b), "x")
x = Px.gen()
fx = x**d - c

for (m, _) in fx.roots():
    flag = long_to_bytes(int(m))
    if b"amateursCTF" in flag:
        print(flag)
        exit()

# amateursCTF{w3ll_m4yb3_if_th3r3_w3r3_n0_fl4g_to_st3aL_n0_on3_w0uld_st3al_1t!!!!!_lmao_60b15b45}
```
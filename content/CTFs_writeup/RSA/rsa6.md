---
title: "RSA6"
---

We are given a file called `gen.py` in which we can see the following code:

```python
from Crypto.Util.number import getStrongPrime, isPrime, inverse, bytes_to_long as b2l

FLAG = open('flag.txt', 'r').read()

# safe primes are cool 
# https://en.wikipedia.org/wiki/Safe_and_Sophie_Germain_primes
while True:
    q = getStrongPrime(512)
    p = 2*q + 1
    if (isPrime(p)):
        break

n = p*q
phi = (p-1)*(q-1)
e = 0x10001
d = inverse(e, phi)

pt = b2l(FLAG.encode())
ct = pow(pt,e,n)

open('output.txt', 'w').write(f'e: {e}\nd: {d}\nphi: {phi}\nct: {ct}')
```

as we can see, we are given e, d, phi and ct. The values are:
```
e: 65537
d: 53644719720574049009405552166157712944703190065471668628844223840961631946450717730498953967365343322420070536512779060129496885996597242719829361747640511749156693869638229201455287585480904214599266368010822834345022164868996387818675879350434513617616365498180046935518686332875915988354222223353414730233
phi: 245339427517603729932268783832064063730426585298033269150632512063161372845397117090279828761983426749577401448111514393838579024253942323526130975635388431158721719897730678798030368631518633601688214930936866440646874921076023466048329456035549666361320568433651481926942648024960844810102628182268858421164
ct: 37908069537874314556326131798861989913414869945406191262746923693553489353829208006823679167741985280446948193850665708841487091787325154392435232998215464094465135529738800788684510714606323301203342805866556727186659736657602065547151371338616322720609504154245460113520462221800784939992576122714196812534
```

after a bit of analysis, it's clear that we have to find n, and since we know from the source code that ```p = 2*q + 1```, we can say that  
```φ(n) = (2*q + 1 - 1) * (q - 1) = 2*q*(q - 1)  => q*(q - 1) = φ(n) / 2```  

we can use the following script to find q and then n:  
```python
from Crypto.Util.number import long_to_bytes
from gmpy2 import iroot
from sympy.solvers import solve
from sympy import Symbol


e= ...
d= ...
phi= ...
ct= ...

# phi(n) = (2*q + 1 - 1) * (q - 1) = 2*q*(q - 1)
# => q*(q - 1) = phi(n) / 2

q = Symbol('q')
val_q = solve(q**2 - q - phi//2, q)[1]

p = 2*val_q + 1
n = int(p*val_q)
# print(len(bin(n)[2:]))


print(long_to_bytes(pow(ct, d, n)))
# flag{8b76b85e7f450c39502e71c215f6f1fe}
```
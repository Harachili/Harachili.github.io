p and q generated with polynomials with a factor k in common

We are given this code, in which we can see how p and q are generated:

```python
#!/usr/bin/env python3

from Crypto.Util.number import *
from flag import flag

def keygen(nbit = 64):
	while True:
		k = getRandomNBitInteger(nbit)
		p = k**6 + 7*k**4 - 40*k**3 + 12*k**2 - 114*k + 31377
		q = k**5 - 8*k**4 + 19*k**3 - 313*k**2 - 14*k + 14011
		if isPrime(p) and isPrime(q):
			return p, q

def encrypt(msg, n, e = 31337):
	m = bytes_to_long(msg)
	return pow(m, e, n)

p, q = keygen()
n = p * q
enc = encrypt(flag, n)

print(f'n = {n}')
print(f'enc = {enc}')

```

And in the file output.txt we have the values of n and enc, which are:
```
n = 44538727182858207226040251762322467288176239968967952269350336889655421753182750730773886813281253762528207970314694060562016861614492626112150259048393048617529867598499261392152098087985858905944606287003243
enc = 37578889436345667053409195986387874079577521081198523844555524501835825138236698001996990844798291201187483119265306641889824719989940722147655181198458261772053545832559971159703922610578530282146835945192532
```
As we can notice, there is a factor k in common in the generation of both p and q. The first solution I wrote worked with the idea of finding k using classic binary search and looking when I get n by simply checking whether  
p(k)*q(k)-n==0:

```python
from Crypto.Util.number import long_to_bytes
from math import gcd

def p(k):
  return k**6 + 7*k**4 - 40*k**3 + 12*k**2 - 114*k + 31377
def q(k):
  return k**5 - 8*k**4 + 19*k**3 - 313*k**2 - 14*k + 14011
def r(k):
  return p(k)*q(k)-n
def avg():
  return (min_i+max_i)//2

n = ...
c = ...
e = 31337

min_i = 0
max_i = 18446744073709551615 # 2**64 - 1
k = avg()
t = r(k)

while t != 0:
  k = avg()
  t = r(k)
  if t > 0:
    max_i = avg()
  elif t < 0:
    min_i = avg()


p, q = p(k), q(k)
print(gcd(p,q))
assert n == p*q
phi = (p-1)*(q-1)
d = pow(e, -1, phi)
m = pow(c, d, n)
print(long_to_bytes(m).decode())

# CCTF{F4C70r!N9_tRIcK5_aR3_fUN_iN_RSA?!!!}
```

Another easy approach would be to find the roots of the equation resulting from multiplying the two polinomials p(k) * q(k) subtracting the value of n. Here is the code:

```python
from Crypto.Util.number import long_to_bytes as ltb
from sympy import *

k=var('k')

enc = 37578889436345667053409195986387874079577521081198523844555524501835825138236698001996990844798291201187483119265306641889824719989940722147655181198458261772053545832559971159703922610578530282146835945192532
n = 44538727182858207226040251762322467288176239968967952269350336889655421753182750730773886813281253762528207970314694060562016861614492626112150259048393048617529867598499261392152098087985858905944606287003243
e = 31337
pPol = k**6 + 7*k**4 - 40*k**3 + 12*k**2 - 114*k + 31377
qPol = k**5 - 8*k**4 + 19*k**3 - 313*k**2 - 14*k + 14011
nPol = pPol * qPol - n

find_p_Pol = lambda k: k**6 + 7*k**4 - 40*k**3 + 12*k**2 - 114*k + 31377
find_q_Pol = lambda k: k**5 - 8*k**4 + 19*k**3 - 313*k**2 - 14*k + 14011

root = list(roots(nPol).keys())[0]
print("root: ", root)

p = find_p_Pol(k=root)
q = find_q_Pol(k=root)

assert p*q == n

phi = (p-1)*(q-1)
d = pow(e,-1,int(phi))
pt = pow(enc,d,n)
print(ltb(pt))

# CCTF{F4C70r!N9_tRIcK5_aR3_fUN_iN_RSA?!!!}
```
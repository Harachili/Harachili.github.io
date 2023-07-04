256-bit RSA where e^2|p−1 and e^2|q−1. 

Intended solution: factor N, then use sage's nth_root() function to get all candidate decryptions. 
Finally, combine using Chinese Remainder Theorem.

It's simple for e|p−1, but for higher-powers of e involves solving a (small) discrete logarithm problem. 
Fortunately, sage has it implemented as a built-in.

Many resources online describe how to proceed if e | p-1, but they don't describe the general case for higher powers of e.

```python
from Crypto.Util.number import long_to_bytes
from sage.all import *

N = 57996511214023134147551927572747727074259762800050285360155793732008227782157
e = 17
cipher = 19441066986971115501070184268860318480501957407683654861466353590162062492971
# http://factordb.com/index.php?query=57996511214023134147551927572747727074259762800050285360155793732008227782157
p, q = 172036442175296373253148927105725488217, 337117592532677714973555912658569668821

assert p * q == N

# print(mod(cipher, p))

p_roots = mod(cipher, p).nth_root(e, all=True)
q_roots = mod(cipher, q).nth_root(e, all=True)

for xp in p_roots:
    for xq in q_roots:
        x = crt([Integer(xp), Integer(xq)], [p,q])
        x = int(x)
        flag = long_to_bytes(x)
        if flag.startswith(b"dice"):
            print(flag.decode())
# dice{cado-and-sage-say-hello}

```
---
title: "diffhell"
tags: "GPNCTF-2023"
---

This challenges I didn't manage to finish during the CTF, but I found it interesting enough to write about it anyway.  
It gives us this code:

```python
import hashlib

FLAG = b"GPNCTF{fake_flag}"

def genCommMat(M): 
    u = M[0][0]
    v = M[0][1]
    x = M[1][0]
    y = M[1][1]
    a = M.base_ring().random_element()
    b = M.base_ring().random_element()
    R = M.base_ring()
    c = b*x * v^-1
    d = (a*v + b*y - b*u)*v^-1
    return Matrix(R,[[a,b],[c,d]])

def genGLM(K):
    a,b,c,d = [K.random_element() for _ in [0,0,0,0]]
    M = Matrix(K,[[a,b],[c,d]])
    return M if M.rank() == 2 else genGLM(K)


#starting flag transmission
p = random_prime(2**41,2**42)
A = GL(2,GF(p)).random_element().matrix() # group of automorphisms (i.e. invertible endomorphisms), General linear group
B = genCommMat(A)
G = GL(2,GF(p)).random_element().matrix()
print("Welcome to Dr. Meta's Office. Leading villan since 1980")
print(p)
print("Due to some construction issues there is some information leaking")
print("But rest be assured, to secure his evil plans Dr. Meta has refurbished Cryptography to secure his secrets")
print("Sadly Dr. Meta has lost his keys after gluing himself to an exmatriculation form and his keys to the table below... With his exmatriculation in his hand and his keys on the table can you help out Dr. Meta and decrypt his important data?")


print(G)
print("Look something has fallen from the back of the Turing machine")
print(A^-1*G*A)
gA   = A^-1 * G * A
gB   = B ^-1 * G * B 
print("Look we found this on a stack of 'rubbish'")
print(gB)

# B^-1 * A^-1 * G * A * B

super_secret_key = B^-1*gA*B
if super_secret_key != A^-1*gB*A:
    print("OHH nooo a MIMA X blew up. The plans to take over the world are destoryed.")

m = hashlib.sha256()
m.update(f"{super_secret_key[0][1]}{super_secret_key[1][0]}{super_secret_key[1][1]}{super_secret_key[0][0]}".encode())
otp = m.digest()
print("Here gulasch spice mix formula to take over the GPN ")
encMsg = [fl^^ot for fl,ot in zip(FLAG,otp)]
print(encMsg)
```

After a brief analysis of it, we can see that there are some interesting leaks, namely `gA = A^-1 * G * A` and `gB = B ^-1 * G * B`. I started thinking, obviously, about how can we retrieve the super secret key from these two matrices, and I came up with this idea:
# I proved that A and B commute (AB=BA), then I thought that since gA, gB and G are similar, 
# then they have the same Jordan form, I tried to calculate it, and indeed they do, so I end up with
# gA = S\*J\*S^-1
# gB = S2\*J\*S2^-1
# G = T\*J\*T^-1
# => S^-1\*gA\*S = J = T^-1\*G\*T
# => gA = S\*T^-1\*G\*T\*S^-1
# By definition gA = A^-1\*G\*A
# => A^-1 = S\*T^-1
# Same reasoning for finding B 

So I wrote this code to find the matrices A and B, but for reasons that still are obscure to me, A and B are indeed right to find gA and gB, but it doesn't pass the check `assert super_secret_key == A^-1*gB*A `, where the super_secret_key's value is `B^-1*gA*B`.  
This is my code:
    
```python
import hashlib


p = 1045113370783   
G = matrix( GF(p), [[629261178046,  98997595995],[485523751128, 286628655675]])
gA = matrix( GF(p), [[ 16103008344, 958954546455],[ 53599593314, 899786825377]])
gB = matrix( GF(p), [[571572197143, 754783882970],[597845308985, 344317636578]])
encMsg = [94, 65, 61, 228, 17, 82, 47, 103, 97, 42, 240, 34, 91, 188, 235, 51, 194, 159, 64, 148, 117, 47, 141, 164, 240, 200, 115, 123, 28, 226, 249, 64]

J, S   = gA.jordan_form(transformation=True)
J1, S2 = gB.jordan_form(transformation=True)
J2, T  = G.jordan_form(transformation=True)

assert J == J1 and J1 == J2, "Not same Jordan form" # passed the check

# I proved that A and B commute (AB=BA), then I thought that since gA, gB and G are similar, 
# then they have the same Jordan form, I tried to calculate it, and indeed they do, so I end up with
# gA = S*J*S^-1
# gB = S2*J*S2^-1
# G = T*J*T^-1
# => S^-1*gA*S = J = T^-1*G*T
# => gA = S*T^-1*G*T*S^-1
# By definition gA = A^-1*G*A
# => A^-1 = S*T^-1
# Same reasoning for finding B 

A = (S*T^-1)^-1

assert A == T*S^-1 

B = (S2*T^-1)^-1
print(A)
print(B)
assert gA == A^-1 * G * A
assert gB == B ^-1 * G * B 

# Step 2: Calculate otp
super_secret_key = B^-1*gA*B
assert super_secret_key == A^-1*gB*A # Doesn't pass this check, the secret key recovered is wrong, I'm puzzled

#########################################################################################################################

otp_string = f"{super_secret_key[0][1]}{super_secret_key[1][0]}{super_secret_key[1][1]}{super_secret_key[0][0]}"
m = hashlib.sha256()
m.update(otp_string.encode())
otp = m.digest()

# Step 3: Decrypt the message
decMsg = [chr(fl ^^ ot) for fl, ot in zip(encMsg, otp)]
print(decMsg)
print(''.join(decMsg))
```

After the competition ended, I looked at others approaches, and no one used Jordan form, so I'm still puzzled about why my approach didn't work. Now I'm gonna paste here a couple of approaches, one which is the intended and which is an unintended solution. The first one goes like this:  
```
Let Q = A^-1 * G * A and R = B^-1 * G* B.
the equalities AQ = GA and BR = GB are each linear systems of 4 variables, and you can solve each system to get the 2-dimensional solution spaces for A and B. We can parameterize any A and B by two variables.
You also need AB = BA. Since A and B are parameterized by 2 variables each, the system AB = BA becomes a system of quadratic equations in the parameters. You can use Sage to solve this system (using Groebner bases to solve systems of polynomials). You still end up with two free parameters in the solution, so you can just set those arbitrarily and get out a valid A and B that satisfy all the properties of the generated matrices. 
```
While the second solution, the unintended one exploits the fact that the function `random_element()` seeds the random generator with the prime `p`, so it's possible to predict the values of the matrices `A` and `B`.  
The code of both are in the following code (for which I thank `kishou yusa` who wrote it):

```python
import hashlib
from pwn import process, remote

ONLINE = True


def get_matrix(io: process, p: int):
    M = [[int(x) for x in io.recvline(keepends=False).decode().lstrip("[").rstrip("]").split(" ") if x != ""] for _ in range(2)]
    return matrix(GF(p), M)


def intended():
    global p, G, gA, gB, flag_enc
    
    F = PolynomialRing(GF(p), [f"A{i + 1}" for i in range(4)] + [f"B{i + 1}" for i in range(4)])
    A, B = matrix(2, F.gens()[:4]), matrix(2, F.gens()[4:])

    polynomials = []
    polynomials.extend([x - y for x, y in zip((G * A).list(), (A * gA).list())])
    polynomials.extend([x - y for x, y in zip((G * B).list(), (B * gB).list())])
    polynomials.extend([x - y for x, y in zip((A * B).list(), (B * A).list())])
    I = F.ideal(polynomials)
    assert I.dimension() == 2

    Q = F.quotient(I.groebner_basis(), [f"A{i + 1}" for i in range(4)] + [f"B{i + 1}" for i in range(4)])
    A, B = A.change_ring(Q), B.change_ring(Q)
    
    F = PolynomialRing(GF(p), ["A4", "B4"])
    # cast to string to avoid the conversion error
    AB = matrix(GF(p), 2, [F(str(ab))(0x1337, 0x1337) for ab in (A * B).list()])

    super_secret_key = AB^-1 * G * AB
    m = hashlib.sha256()
    m.update(f"{super_secret_key[0][1]}{super_secret_key[1][0]}{super_secret_key[1][1]}{super_secret_key[0][0]}".encode())
    otp = m.digest()
    flag = [x ^^ y for x, y in zip(flag_enc, otp)]
    print(bytes(flag))


def unintended():
    global p, G, gB, flag_enc
    # matrix generation is seeded with the prime p
    A = GL(2, GF(p)).random_element().matrix()

    super_secret_key = A^-1 * gB * A
    m = hashlib.sha256()
    m.update(f"{super_secret_key[0][1]}{super_secret_key[1][0]}{super_secret_key[1][1]}{super_secret_key[0][0]}".encode())
    otp = m.digest()
    flag = [x ^^ y for x, y in zip(flag_enc, otp)]
    print(bytes(flag))


#io = remote("diffhell-0.chals.kitctf.de", 1337, ssl=True) if ONLINE else process(["sage", "challenge.sage"])
#io.recvuntil(b"Welcome to Dr. Meta's Office. Leading villan since 1980\n")
#p = int(io.recvline(keepends=False))
#io.recvuntil(b"can you help out Dr. Meta and decrypt his important data?\n")
#G = get_matrix(io, p)
#io.recvuntil(b"Look something has fallen from the back of the Turing machine\n")
#gA = get_matrix(io, p)
#io.recvuntil(b"Look we found this on a stack of 'rubbish'\n")
#gB = get_matrix(io, p)
#io.recvuntil(b"Here gulasch spice mix formula to take over the GPN \n")
#flag_enc = bytes(eval(io.recvline(keepends=False).decode()))
#io.close()

p = 1045113370783   
G = matrix( GF(p), [[629261178046,  98997595995],[485523751128, 286628655675]])
gA = matrix( GF(p), [[ 16103008344, 958954546455],[ 53599593314, 899786825377]])
gB = matrix( GF(p), [[571572197143, 754783882970],[597845308985, 344317636578]])
flag_enc = [94, 65, 61, 228, 17, 82, 47, 103, 97, 42, 240, 34, 91, 188, 235, 51, 194, 159, 64, 148, 117, 47, 141, 164, 240, 200, 115, 123, 28, 226, 249, 64]


intended()
unintended()

# GPNCTF{Dr.M3t4F0rTh3W1n!?0x1337}
```
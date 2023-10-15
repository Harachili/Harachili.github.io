---
title: "cherry-leak"
tags: "TCP1PCTF 2023"
---

In this challenge we are given a python code and a nc connection to a server.  
The python code is the following:

```python
from Crypto.Util.number import getPrime, bytes_to_long

p = getPrime(1024)
q = getPrime(512)
n = p * q
e = 65537

FLAG = b"TCP1P{???????????????????????????????????????}"

lock = False
while True:
    print("1. Get new prime")
    print("2. Get leak")
    print("3. Get flag")
    print("4. Exit")
    print("> ", end="")
    try:
        choice = int(input())
    except:
        break
    if choice == 1:
        if lock:
            print("You can't do that anymore!")
            continue
        print("which prime? (p/q)")
        print("> ", end="")
        prime = input()
        if prime == "p":
            p = getPrime(1024)
        elif prime == "q":
            q = getPrime(512)
        else:
            print("What?")
            continue
        n = p * q
        lock = True
    elif choice == 2:
        print("choose leak p ? q (+-*/%)")
        print("> ", end="")
        leak = input()
        if leak == "+":
            print(f"p + q = {pow(p + q, e, n)}") # nuh uh
        elif leak == "-":
            print(f"{p - q = }")
        elif leak == "*":
            print(f"p * q = {pow(p * q, e, n)}") # nuh uh
        elif leak == "/":
            print(f"p // q = {pow(p // q, e, n)}") # nuh uh
        elif leak == "%":
            print(f"{p % q = }")
        else:
            print("What?")
    elif choice == 3:
        print(f"c = {pow(bytes_to_long(FLAG), e, n)}")
        lock = True
    elif choice == 4:
        break
    else:
        print("What?")
```

And how we can see, we have a server to connect to, from which we can receive a variety of leaks:  

- (p+q)^e mod n
- p-q
- (p\*q)^e mod n
- p//q
- p%q
  
Moreover, we can modify the value of either p or q, but only once.  

These are a lot of leaks to analyze, but most of them are not useful, and it is clear from the fact that they leak (p\*q)^e mod n, which will always be equal to 0.  
So we have to understand on which leaks to focus on. At this point I started analyzing the leaks we have, just to realize that with the possibility of changing either p or q, we can easily use the values `p-q` and `p%q`.  
The procedure is simple: 

- a = (p1 - q) - (p1 % q) = k1*q
- b = (p2 - q) - (p2 % q) = k2*q
- => With high probability GCD(a, b) = q
- Try this as many times as needed, until q is prime lol
  
To solve this I used a sage script, so if you run the following script in python you might have to import gcd and is_prime.  
This is the script:

```python
from pwn import remote
from Crypto.Util.number import GCD, long_to_bytes as l2b

host = "ctf.tcp1p.com"
port = int(13339)

FOUND = False

def send(x):
    r.sendlineafter(b'> ', x.encode())

def rec():
    return int(r.recvline().split(b' ')[-1].strip().decode())

while not FOUND:
    r = remote(host, port)
    # Retrieve p1 - q 
    send("2")
    send("-")
    sub1 = rec()
    # Retrieve p1 % q
    send("2")
    send("%")
    mod1 = rec()
    # Retrieve p2 
    send("1")
    send("p")
    # Retrieve p2 - q
    send("2")
    send("-")
    sub2 = rec()
    # Retrieve p2 % q
    send("2")
    send("%")
    mod2 = rec()
    # Retrieve c
    send("3")
    c = rec()

    # a = (p1 - q) - (p1 % q) = k1*q
    # b = (p2 - q) - (p2 % q) = k2*q
    # => With high probability GCD(a, b) = q
    # Try this as many times as needed lol
    a = sub1 - mod1
    b = sub2 - mod2
    q = GCD(a, b)
    try:
    	assert is_prime(q)
    	FOUND = True
    except:
        r.close()
        continue
    p = sub2 + q
    
    
    n = p*q
    phi = (p-1)*(q-1)
    e = 65537
    d = int(pow(e, -1, phi))
    m = int(pow(c, d, n))
    flag = l2b(m).decode()
    
    print(flag)

# TCP1P{in_life's_abundance_a_fragment_suffices}
```

---
title: "thepassw902jtxjuo90tj65"
tags: "hackcon 2023"
---

This challenge was fun, since it was the first time in which I used lattices to solve an RSA challenge.  
Let's start by looking at the challenge description:

```txt
Factorizing large numbers is too easy, so I thought I'll just give you 1 of the factors
```

As soon as I read this, I didn't even want to download the file, since I thought it was a preliminary challenge just to make people understand how RSA works.  
But then I saw that it had few solves (even though it was the most solved challenge), so I decided to give it a try.  
We can see the chall.py file:
    
```python
from Crypto.Util.number import getPrime, bytes_to_long
from Crypto.Util.Padding import pad, unpad

# how about 2 part flags
flag1 = b"FLAG PART 1 HEHEHE"
flag2 = b"FLAG PART 2 not hehe :("

flagnum = bytes_to_long(flag1)
flagnum2 = bytes_to_long(pad(flag2, 256))

assert len(bin(flagnum)[2:]) == 303

p = getPrime(2048)
q = getPrime(2048)
N = p * q
e = 65537

# p isn't needed anymore; must be corrupted
p = p^flagnum
print("masked_p={}".format(hex(p)))
print("N={}\ne={}".format(hex(N), hex(e)))

ct = pow(flagnum2, e, N)
print("ct={}".format(hex(ct)))
```

So as we can see, we have two parts of the flag: the first part is long 303 bits and it gets xored with the factor p, which then is leaked, while the second part is simply encrypted with RSA.  
Ok, looks clear that there has to be a way to recover the first part of the flag, but how?  
Looking a bit on the internet, I attempted different solutions, but none of them worked.  
Then I found [this](https://eprint.iacr.org/2020/1506.pdf) paper which explains how to factorize from consecutive bits of p.  
Bingo! I started implementing the paper and it successfully worked, this is the final script I used:

```python
from Crypto.Util.number import long_to_bytes as ltb

masked_p = 0x9ac12962669f9be2f8e6edbca1e6c307b0d0cf2432c4afdeefaa0fd7d106964056a326f8af960ad4e5aa392625f67d46d05c3dbf82b8d508cf4c9369faf7c6a36ffd4028de967424f75c880042bfcf8e731f67629f600f1b4bf27476599aaaed866f7c2e2d2a52fe6b4444b1d2ae67af561829db0229883f23b880dd326df8d38eeeb3f8f7f0d3c4b357175f219f3dad751855e011b9dd088bb046ca1a3943808c19daedf014245a8e165854931eff3a56b57c0adf9546550ac473e29e10b077ce8f6ac2e044d853ab9662ba8bba387afd29355c8d1a0ce96ea5cad161b25ddc34dd43724ce6977916379ef35038a82a1c5c8cbf8ee333ba76ed3150e1b626c0
N = 0x8c69cb58bd9b5f6cfbe5973aadad07e20f117333cde4650bafaa76fdb888436d60c61bc883e9bead6edee6d8323fa344a0dec482100c826ada5e3c970e6f017faa25e06680d36ceb2fc53b0d4065a5ef411d595a8ae749e01dc9eea1a79a6da2ec4c9428d3ef1577cc2c8ab48fe2d2a19313dc92c6ff2aaa36f0122f047c8fd2793fa10f09955a8473b397983774acc1f64b105adc5adad6a1edfba78fd80d0b04476ab24b60710c3c20e9c3914456d1425b34449cd81bf01d43bb3d9bf3a100dd599443eb94aa5f7d884889585d7f0008c1bbe1403fd9f17a3c5901327fb5c66c04af09d134327dfc3222359e3249bb8309c93f74dc93d981f72e99bde0124800807878f04c5aaebff4a3236d29abbc98b16b0c0657d1e905888cbccc8c0306d7c576f5386068eedcb51753ae20cd314199ab144bcf994d3f015a05e4735dd2d2301ed904a13ce763413b3f007ff3c25ba30e6e809522574c8c37d3c8b1fe7c5de75c8162f42051a062384f237cea5cab45f40fc38e28ebd949de6352f6e289ed855c0aec761a886cb0a6916067d77947d42e4bb6d0b399ca0502776ba0c847831fd07e721a2a01466f4b42bd2ed82bdc3e39637862902b8c3d8cc302d20180bb503a01eac75f433d3d601792c1f4a6e1c62e6d83b3d37cb0d200ee3e6b9463b90b43064d902d93f8578e3aaaa76a356d08d25965ada3f2af3c0a55e20580af
e = 0x10001
ct = 0x79481001fe7aa55c996e3cf84f9e675842b515eab2c30b43c210206235116c9525ff78c6fdc26699d7f680b4b779d792cac8cdbbf54e39cbf95260bb669d6cd1591cd131a7ad4d36183c7970bd491851943366ef960989541912cc317aedf576bc0f10ed3b0d3fd3b6a2e2b706a7c927c04c8282ada3f1c762dd01bdd87ca3e58e38365982e96ac8d7a9365bd9af556f6f1b45ccfc3840e95445247fae08f427763e0ce16788539f84822e25070414bd559b195e6f63a66d7289140bd40f0d7017ef93a29d42957de441d5868b13678d3670c2e33bf16bcb5d0c1f8f2c17c6c2defd3820cb89d13af09adba0130c3485f14fc6f5c3ca40554d322a6b222ccc95facf77510245431479dddc98c5f7624539a1850aaa9d3447576a7c8ea51e2b573fd8c5358315caf6b83265631897abbb9f0794174b64731c3e6e652453c0fa837b78d8d57e064e27c745272ad7d255ff5123dd0f8466a4a5f9587508baa47f1cbe2cf19c65c345cdc91488fb6849fbd7029d6aef964d407e810ac23225555d55ce539c3bb0ca09897ddbbf647fd39e1521dd7d0a3b70f22d3a1d4a4261422397a553dce42e821d140ae48e3bb1a758054ced96dc729bb02e027a043464ed71717441417f4ca2a050b83f4911fd429cb69c00ad043303b9f9966d26ad9dca3788d5870c78abafddbea8bd1beec7e60b86daf5f2b2cba49b34430664ba893ac0e0

# https://eprint.iacr.org/2020/1506.pdf

# Number of leaked bits
R = 2^303

# Build the matrix as written in the paper
B = matrix([[R^2, R*masked_p, 0], [0,R,masked_p], [0,0,N]])

# Calculate LLL and take the first minimal solution vector 
# (in this case there were 2)
lattice = B.LLL()[0] 

assert lattice[0] % R^2 == 0
assert lattice[1] % R == 0

x = var('x')
f = lattice[0]/R^2 * x^2 + lattice[1]/R * x + lattice[2]

mask = f.roots()[0][0]

p = gcd(masked_p+mask,N)

assert gcd(p, N) == p

first_part_flag = ltb(masked_p^^p)

q = int(N) // int(p)
phiN = (p-1)*(q-1)
d = pow(e,-1,phiN)

second_part_flag = ltb(int(pow(ct,d,N)))

print(first_part_flag + second_part_flag)

# d4rkc0de{uhh_1s_thi5_inn0vative_et_al?ig_n0t_hehe}
```


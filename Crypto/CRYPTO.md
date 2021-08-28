# OnHacking | Crypto

Crypto is considered one of the harder categories, obviously due to the math involved. Knowing math is required, just as knowing how the web works for web, or assembly for RE & pwn. While good google search queries can help people (and by people I mean everyone) deal with this category, it is also important to put in the effort (if it is a category you want to get into) to gain the mathematical knowledge needed to understand what is going on. We are going to split it into these areas:
* [Classic Cryptography](#classic-cryptography)
* [Modern Cryptographhy](#modern-cryptography)
* [Everything Else](#everything-else)

But first, some

# Python Basics

Python is generally the goto language for Crypto mainly because of it's overall strong points. Multiple libraries,

```
# Integers & ASCII
>>> ord('A')
65
>>> chr(65)
'A'

# List of int ASCII values
>>> [ord(i) for i in word]

# Integers & Hex
>>> hex(13)
'0xd'
>>> int('d', 16)
13
>>> 0xd
13

# Strings & Bytes
>>> bytes('Hello', 'utf-8')
b'Hello'
>>> b'Hello'.decode('utf-8')

# Bytes & Hex
>>> b'Hello'.hex()
'48656c6c6f'
>>> bytes.fromhex('48656c6c6f')
b'Hello'

# Long & Bytes
>>> from Crypto.Util.number import long_to_bytes, bytes_to_long
>>> bytes_to_long(b'Hello')
310939249775
>>> long_to_bytes(310939249775)
b'Hello'


```

# Classic Cryptography
Classic Cryptography isn't used today, it's mostly kept for brain puzzles. You can easily find them in CTFs.


# Modern Cryptography

## RSA

### Exponent

If the public exponent is low, then maybe wheh pow(ct, e) happens, the result isn't larger than n, som pow(ct, e, n) = pow(ct, e). In this case, we just have to calculate the e-th root of pow(ct, e). I recommend gmpy2.iroot.

# Everything Else
What a name! The name is self explanatory. Here belongs everything that can be found in CTF Crypto challenges, like abusing pseudo-random generators, hashes,
XOR, etc..

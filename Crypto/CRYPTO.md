# OnHacking | Crypto

Crypto is considered one of the harder categories, obviously due to the math involved. Knowing math is required, just as knowing how the web works for web, or assembly for RE & pwn. While good google search queries can help people (and by people I mean everyone) deal with this category, it is also important to put in the effort (if it is a category you want to get into) to gain the mathematical knowledge needed to understand what is going on. We are going to split it into these areas:
* [Classic Cryptography](#classic-cryptography)
* [Modern Cryptographhy](#modern-cryptography)
* [Everything Else](#everything-else)

But first, some

# Python Basics


# Classic Cryptography
Classic Cryptography isn't used today, it's mostly kept for brain puzzles. You can easily find them in CTFs.


# Modern Cryptography

## RSA

### Exponent

If the public exponent is low, then maybe wheh pow(ct, e) happens, the result isn't larger than n, som pow(ct, e, n) = pow(ct, e). In this case, we just have to calculate the e-th root of pow(ct, e). I recommend gmpy2.iroot.

# Everything Else
What a name! The name is self explanatory. Here belongs everything that can be found in CTF Crypto challenges, like abusing pseudo-random generators, hashes,
XOR, etc..

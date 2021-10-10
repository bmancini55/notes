# ECDSA

```
m = message
z = hash(m)
e = private key
G = generator point
k = random nonce

P = eG = public key
u = z/s
v = r/s
R = kG
r = x-coord of kG

u*G + v*P = R
u*G + v*P = k*G
u*G + v*e*G = k*G
z/s*G + r/s*e*G = k*G

s = (z+r*e)/k

signature = (s, r)
```

# Schnorr

- deleloped in 1991
- patent expired in 2008
- simpler
  - no division (in comparison to ECDSA and Elgamal)
- breaking Schnorr breaks the discrete log problem

```
s = k + e*z
```

### Multiple signatures

```
P1 = e1*G
s1 = k1 + e1*z

P2 = e2*G
s2 = k1 + e2*z

# simply add them up
s1+s2 = (k1+k2) + (e1+e2)*z

# same as the single variant, except we add the terms
s' = k' + e'*z

# can easily create the public key by adding them
P' = (e1 + e2)*G
```

### Pay to Contract

# Resources

- [Video - Introduction to Schnorr Signatures with Elichai Turkei](https://www.youtube.com/watch?v=XKatSGCZ-gE)

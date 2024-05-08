# Introduction to cryptography - RSA

It has been a while since I've written a blogpost, but I find it interesting that everyone hangs out at [RSA conference](https://www.rsaconference.com/usa) but so few know how RSA really works.  
In this blogpost I'll try to explain why RSA is so powerful, as well as explain the math behind its basic form.  

## Motivation
Until this point, we have described several cryptosystems that use the same key for both encryption and decryption. Those are known as `Symmetric encryption` schemas.  
The great disadvantage is that two parties (traditionally, `Alice` and `Bob`) have to exchange a secret somehow (the key), which is a non-trivial task assuming distances.  
For instance, when you browse, you most commonly use the `HTTPS` protocol, which is just `http` over `TLS`. However, you never "met" with the web server and exchanged a secret.  
This emphasizes the necessity of exchanging secrets securely without physically meeting, and while key-exchange is a slightly different problem, it's completely solvable assuming an `Asymmetric encryption` schema.  
In short, an `Asymmetric encryption` is a cryptosystem in which decryption and encryption use *different keys*. Traditionally we call one a `private key` and that is known only to one party (who *owns* the key) and the other is known as a `public key`, which is, well, *public*.  
Assuming such a system, it's easy to exchange a secret:

1. `Alice` publishes her `public key`, anyone is able to use it, but keeps her `private key` to herself.
2. `Bob` creates a random secret and uses `Alice`'s `public key` to encrypt it.
3. `Bob` sends encrypted secret to `Alice`, and `Alice` decrypts it with her `private key`.
4. Now `Alice` and `Bob` both know a joint secret, and can start using `Symmetric encryption`.

The main takeaway from these steps is that `Bob` sends the encrypted secret, and because *only Alice has the private key, only Alice can decrypt the message*.
There are several challenges with that (e.g. how does `Bob` know that the `public key` indeed belongs to `Alice`) but those are solvable (in some sense, I will discuss that in a later blogpost).  
The important part is that `Asymmetric encryption` could be used for key-exchanges.  
Of course, the main challenge is - how to create such a cryptosystem? Well, using math.

## The algorithm
RSA works within the multipicative [group](https://en.wikipedia.org/wiki/Group_(mathematics)) `mod n`, which contains all the [coprimes](https://en.wikipedia.org/wiki/Coprime_integers) to `n`, with a multipication operation `mod n`.  
If you don't know what those are then I refer you to my previous blogposts, and specifically [modular arithmetics](https://github.com/yo-yo-yo-jbo/crypto_modular/) which we will be using a lot today.  
Well, how does the algorithm work? Let us first describe the algorithm before explaining how it works:

1. `Alice` creates two random large primes, `p` and `q` (how to create a "random" prime? I will discuss that in a later blogpost), and defines `n = p*q`.
2. `Alice` uses a number that we'll call `e` (that is not *the* [e](https://en.wikipedia.org/wiki/E_(mathematical_constant)), remember, we are working with integers `mod n`!). In reality, `e` is really constant and "baked into" the cryptosystem (commonly `65537`, I will explain why later).
3. `Alice` calculates a number `d` such that `d` is the multipicative inverse of `e` mod `phi(n)`, where `phi` is the [Euler Totient Function](https://en.wikipedia.org/wiki/Euler%27s_totient_function). Alice could calculate `phi(n)` directly: `phi(n) = (p-1)(q-1)`.
4. `Alice` publishes her `public key = (e, n)` and keeps her `private key = (d, n)`, and gets rid of `p` and `q`.
5. At this point, for every `x` in the multipicative group `mod n`, the following is true: `(x^d)^e = (x^e)^d = x (mod n)`, where `^` is exponentiation.
6. When `Bob` wants to use `Alice`'s `public key` to encrypt such a value `x`, all he does is send `x^e (mod n)`.
7. Alice can then decrypt `Bob`'s message by raising it to the `d`-th power (`mod n`), thus getting `x` back.

This sounds rather complicated but is actually quite simple. Let us explain how values are picked.

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
RSA works within the multipicative [group](https://en.wikipedia.org/wiki/Group_(mathematics)) `mod n`, which contains all the [Coprimes](https://en.wikipedia.org/wiki/Coprime_integers) to `n`, with a multipication operation `mod n`.  
If you don't know what those are then I refer you to my previous blogposts, and specifically [modular arithmetics](https://github.com/yo-yo-yo-jbo/crypto_modular/) which we will be using a lot today.  
Well, how does the algorithm work?

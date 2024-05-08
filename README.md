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

1. `Alice` creates two random large primes, `p` and `q` (how to create a "random" prime? I will discuss that in a later blogpost), and defines `n = p*q`. Note that "large" is a relative term, today `2048` bit values are considered "safe" (but not [Quantum resilient](https://en.wikipedia.org/wiki/Post-quantum_cryptography)).
2. `Alice` uses a number that we'll call `e` (that is not *the* [e](https://en.wikipedia.org/wiki/E_(mathematical_constant)), remember, we are working with integers `mod n`!). In reality, `e` is really constant and "baked into" the cryptosystem (commonly `65537`, I will explain why later).
3. `Alice` calculates a number `d` such that `d` is the multipicative inverse of `e` mod `phi(n)`, where `phi` is the [Euler Totient Function](https://en.wikipedia.org/wiki/Euler%27s_totient_function). Alice could calculate `phi(n)` directly: `phi(n) = (p-1)(q-1)`.
4. `Alice` publishes her `public key = (e, n)` and keeps her `private key = (d, n)`, and gets rid of `p` and `q`.
5. At this point, for every `x` in the multipicative group `mod n`, the following is true: `(x^d)^e = (x^e)^d = x (mod n)`, where `^` is exponentiation.
6. When `Bob` wants to use `Alice`'s `public key` to encrypt such a value `x`, all he does is send `x^e (mod n)`.
7. Alice can then decrypt `Bob`'s message by raising it to the `d`-th power (`mod n`), thus getting `x` back.

This sounds rather complicated but is actually quite simple. Let us explain how values are picked.

## The math
So, why does `RSA` work? This question could be answered if we understand how the values are picked.
1. As I said, `p` and `q` are chosen *randomly*. There are good well-known algorithms for creating a random prime numbers (they are not trivial), but let us assume that is known. Therefore, `n = pq` simply ensures that `n` is a *composite* number but is computationally hard to factorize.
2. The value of `phi(n)` is simply the number of elements in the multipicative group `Z*n`, and is easy for `Alice` to calculate but hard for anyone else. Since the Euler Totient function is multipicative, `phi(n) = phi(p) * phi(q)`, and since for every prime `phi(p) = p-1` we get `phi(n) = (p-1)(q-1)`. Note that an attacker that can calculate `phi(n)` can crack `RSA`; it is believed today that the problem of calculating `phi(n)` is equivalent to the problem of the factorization of `n`, and *that* is believed to be a computationally difficult problem (for a large value of `n`).
3. As I mentioned, `e` is really constant, but it has to be coprime to `phi(n)`. In most cases it'd be `65537`, since it's a prime number it's very likely for it to be a coprime of `phi(n)`, as well as the fact that it's a power of two plus one (`65537 = 2^16+1`), so it's very efficient to use it in exponentiation. I have seen cases where `e=3`, but it's not commonly used and might actually pose security issues, for example, for small values of `x` it's trivial to get `x` out of `x^e (mod n)`.
4. The value of `d` can be efficiently determined using the [Extended Euclidean algorithm](https://en.wikipedia.org/wiki/Euclidean_algorithm), since `Alice` knows `phi(n)`. Again keep in mind that an attacker that can determine `phi(n)` can conclude `d` from it and therefore break the entire cipher.

Why does `(x^d)^e = (x^e)^d = x (mod n)`?
Well, according to [Lagrange](https://en.wikipedia.org/wiki/Lagrange%27s_theorem_(group_theory)), every the size of a `sub-Group` divides the size of the original Group. For example, the only possible sub-Groups of a Group of size `15` must be of sizes `3` or `5` (without including the trivial cases of `1` and `15`).  
Now, let's think of some `x` that belongs to a finite Group `G`, and let's start looking at its powers: `x^0, x^1, x^2, x^3, ...`. Since this can go on forever but the size of `G` is finite, it means that at a certain point we'll get `x^a = x^b` for some integers `a` and `b`. This means that `x^(a-b) = 1` (within Group `G` and its operation), and that means that there's some power of x `c` such that `x^c = 1`.  
The lowest integer (excluding `0`) that maps `x^c` to `1` is called `the order of G` and marked as `c = ord(G)`.  
Well, the order of a Group always divides the Group size, which means `x^n = x^(cm) = (x^c)^m = 1` for some integer `m`.  
To conclude, for every Group `G` with `k` elements, if `a = b (mod k)` then `x^a = x^b`. In our case:
- `k` is really `phi(n)`.
- `ed = 1 (mod k` (because that's how we chose `d`).
- Therefore, `(x^e)^d = x^(ed) = x^1 = x`.

## Why not using just RSA then?
Why should `Alice` and `Bob` not just continue conversing over `RSA`? Because there are several disadvantages:
1. `RSA` is computationally expansive. Raising large numbers to the power of large numbers `mod` large numbers takes non-trivial processing power.
2. You can't use any `x` as input to `RSA` - you can only use `x` that is coprime to `n` - that is a *very large set* but not arbitrarily large.
3. Key sizes for securing `RSA` are huge compared to `symmetric ciphers` - common `AES` (symmetric) key is around `256` bits, while modern `RSA` requires around `2048` or even `4096` bits.
4. Unlike symmetric ciphers, that are relatively okay, `RSA` is not Quantum-resistent. This is not a Quantum computation blogpost (and I am not an expert on the matter), but using [Shor's algorithm](https://en.wikipedia.org/wiki/Shor%27s_algorithm) *assuming a Quantum computer* (which is a big assumption nowadays) is very promising for breaking `RSA` by means of factorization; on the other hand, symmetric ciphers are quite safe - the best known Quantum attack against them is [Grover's algorithm](https://en.wikipedia.org/wiki/Grover%27s_algorithm) which can be solved by just doubling the size of the (already small) symmetric key size.

## Summary
In this blogpost I've described how *basic* RSA works; in truth, there are more sophisticated versions of RSA but they still use the same fundemental principals.  
There are many uses for RSA even today, and I wouldn't exaggarate if I said it's one of those fundemental algorithms that make the internet as we know it today, possible. It's also used extensively for digital signatures, which I plan on covering in the future.  
I still owe a lot of future blogposts - from attacks on RSA to interesting uses, as well as random prime number generation, there's a lot to cover!

Stay tuned!

Jonathan Bar Or

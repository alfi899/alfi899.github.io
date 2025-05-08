---
title: "Network coding in P2P on Basis of ElGamal Encryption"
date: 2019-09-25
categories: [Project]
tags: [Networking, Cryptography, Python]
description: This is a small Project i made for my Bachelor. It combines Network Coding
             and ElGamal Encryption to build a secure File-Sharing application.
pin: false
math: true
mermaid: true
#image:
#  path: /assets/img/posts/htb-machine-name/banner.png
#  alt: "HTB Machine Banner"
#  width: 800
#  height: 400
---

> **Bachelor Project**  
> Secure File-Sharing with Network Coding and ElGamal Encryption

---

## The Idea
Imagine a world where file sharing isn't just fast and decentralized—but also secure by design, resilient to disruption, and 
mathematically elegant. That’s the core idea behind combining peer-to-peer (P2P) networking, network coding, ElGamal encryption, 
and matrix algebra into a single secure file-sharing architecture.

### Why P2P?
A peer-to-peer (P2P) network removes central servers from the equation. Peers (users) cooperate to store, share, and distribute data among themselves. This architecture naturally offers Decentralization — no single point of failure, Scalability — more users means more bandwidth and resources and Redundancy — multiple peers can hold parts of a file.
But decentralization comes with a tradeoff: how do you ensure security and data integrity when nodes are untrusted?
Further we have to think about another point, which is efficency. Thats where Network Coding comes in place.

### Network Coding
In traditional computer networks, routers and switches follow a simple rule: forward packets exactly as they are received. This 
"store and forward" approach works well—but as networks grow in complexity and size, it becomes increasingly inefficient. This is 
where network coding comes in.
Network coding is a communication technique that allows nodes within a network to not only forward packets, but also process and 
combine them before passing them along. Instead of sending each data packet individually, intermediate nodes can mix
incoming data to produce new packets that still preserve the original information.
For that we use `linear algebra`.

Let's view a simple example. Imagine a sender wants to transmit two packets: 

- A = 1010
- B = 1100

to two receivers through a single intermediate node. The node forwards A, then B. Both packets must be transmitted individually.
If we use Network Coding, we could use an XOR operation at the intermediate node(s).
The node computes `A XOR B = 0110` and sends it. If Receiver 1 already has A, it can compute `B = (A XOR B) XOR A`.
Likewise, Receiver 2 can compute A if it already has B.
That means, one coded packet can serve multiple receivers more efficiently.

![network coding](/assets/img/elgamal/Butterfly_network.svg.png)

At its core, network coding turns messages into vectors and treats forwarding as an algebraic operation. Intermediate nodes 
compute linear combinations of these message vectors, like so:

```text
C1 = a1 * M1 + a2 * M2 + ... + an * Mn
```
At the receiving end, once enough independent combinations are received, the original messages can be recovered by solving a 
system of linear equations.
This approach:
- Increases throughput in multicast networks
- Improves robustness against packet loss
- Enables distributed cooperation among nodes

Network coding becomes even more powerful when combined with cryptographic techniques like ElGamal encryption, which supports 
`homomorphic operations`. Which is an essential part that this approach works.

### ElGamal Encryption
Enter ElGamal, a public-key encryption scheme known for its homomorphic property—specifically, it supports multiplicative 
homomorphism. That means:
>If you encrypt two messages and multiply their ciphertexts, the result is an encryption of the product of the original messages.

Let's brakedown how it works.

#### Key Generation
To start using ElGamal, each user generates a key pair—a public key to share with others, and a private key to keep secret.
Here’s how it works:

1. Pick a large prime number `p` and `a` generator `g` of the multiplicative group modulo `p`.

2. Choose a private key `x`, a random number such that:

$$
\begin{equation}
  1 \leq x \leq p-2
\end{equation}
$$

3. Compute the public key component `y`:

$$
\begin{equation}
  y = g^x \mod p
\end{equation}
$$

And so we got two Key's. `Public key: (g, y, p)` and `Private key: x`

#### Encryption
To encrypt a message `m` using the recipient's public key (g, y, p): 

1. Choose a fresh random number r for each encryption (this ensures randomness and security).

2. Compute the ciphertext as a pair `(c1, c2)`:

$$
\begin{equation}
  c_1 = g^r \mod p
\end{equation}
$$


$$
\begin{equation}
  c_2 = y^r \cdot m  \mod p
\end{equation}
$$



So the ciphertext is: 
$$
\begin{equation}
  (c_1, c_2) = (g^r, y^r \cdot m) \mod p
\end{equation}
$$

#### Decryption
To decrypt the ciphertext `(c1, c2)` using the private key `x`, perform the following:

1. Compute:

$$
\begin{equation}
  m = c_2 \cdot c_1^{-x} \mod p
\end{equation}
$$

ElGamal ensures Randomness because each encryption uses a new `r`, making ciphertexts unique.
It also supports multiplicative homomorphism, which means you can multiply ciphertexts and still get meaningful decrypted 
results—crucial for advanced techniques like network coding and secure data aggregation.

![el gamal](/assets/img/elgamal/p2p.png)



## Combining ElGamal and Network Coding
Now that we understand both network coding and ElGamal encryption, let’s see how they can be combined into a powerful method for 
secure, distributed file sharing. The idea is to preserve the algebraic structure of network coding while keeping all data
encrypted using the multiplicative homomorphism of ElGamal.

Thanks to the homomorphic properties, we can multiply two ciphertexts, and this operation will carry over to the encrypted 
messages themselves:


$$
\begin{equation}
  (c_1, c_2)^\alpha \cdot (c_1, c_2)^\beta \rightarrow (g^{r_1\alpha+r_2\beta}, y^{r_1\alpha+r_1\beta} \cdot m_1^\alpha \cdot m_2^\beta)
\end{equation}
$$

This equation shows that we can mix encrypted packets together using scalar coefficients `α`, `β`, without ever revealing the 
plaintexts `m₁` or `m₂`.
In a network coding setup, each peer combines incoming packets using a set of coefficients, and those coefficients can be stored 
in a matrix. If we gather enough combined ciphertexts from the network, we get a system of equations that looks like this:

$$
\begin{equation}
  C = A \cdot M
\end{equation}
$$


Where `C` is a vector of combined ciphertexts, `A` is the coefficient matrix (known to the receiver) and `M` is the original 
message vector (encrypted).

To recover the original encrypted messages, the receiver computes the modular inverse of matrix A modulo q:

$$
\begin{equation}
  M = A{-1} \cdot C \mod q
\end{equation}
$$

After computing this, the encrypted messages can be decrypted as usual using ElGamal's decryption formula.

## Example
Let’s walk through a concrete example of how encrypted messages can be securely combined and reconstructed using ElGamal 
encryption and network coding.

Set up the variables `p=53147 and q=26573`, the Generator `g=7`. The public key component `y=38268`. The Message to encrypt is 
`m₁ = 72 with randomness r₁=9`and `m₂ = 101 with randomness r₂ = 7`.

We encrypt the message using ElGamal:

- E(72) = (14722,38849)
- E(101) = (39200,733)

Now we create combinations of these ciphertexts using exponents (a, b, c, d) to simulate network coding:

Combination 1:

$$
\begin{equation}
  a = 3, b = 5
\end{equation}
$$

$$
\begin{equation}
  (14722,38849)^3 \cdot (39200,733)^5 = (24887,43150)
\end{equation}
$$

Combinaiton 2: 


$$
\begin{equation}
  c = 6, d = 4
\end{equation}
$$

$$
\begin{equation}
  (14722,38849)^6 \cdot (39200,733)^4 = (35968,22424)
\end{equation}
$$

Now we have two coded ciphertexts that are combinations of the original encrypted messages.

We now can decrypt the resulting ciphertexts:

-  X₁=decrypt(24887,43150)=46819
-  X₂=decrypt(35968,22424)=41496


These are linear combinations of the original messages, not the original messages themselves.

We now solve a system of linear equations using matrix algebra.

Our coefficients from the combinations were:

$$
\begin{equation}
  A=\begin{bmatrix}
3 & 5 \\
6 & 4\\
\end{bmatrix}
\end{equation}
$$

We compute its modular inverse modulo `q = 26573`:


$$
\begin{equation}
  A^{-1} \mod q =\begin{bmatrix}
11810 & 25097 \\
8858 & 22144\\
\end{bmatrix}
\end{equation}
$$

We use this inverse to "undo" the network coding and recover the original encrypted messages.


$$
\begin{equation}
  m_1 = X_1^{B[0][0]} \cdot X_2^{B[0][1]} \cdot \mod p = 53057 \mod p = 72
\end{equation}
$$


$$
\begin{equation}
  m_1 = X_1^{B[1][0]} \cdot X_2^{B[1][1]} \cdot \mod p = 101 
\end{equation}
$$

For `m₁` we do one final step 

$$
\begin{equation}
  m_1 = p - 53057 = 72 
\end{equation}
$$

We successfully recovered `m₁=72` and `m₂=101`.



## Python Code
You can see the python code at my GitHub repo. 
```text
https://github.com/alfi899/FileSharing-ElGamal-NetworkCoding
```

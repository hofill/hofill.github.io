---
layout: post
author: hofill
title: Don’t chat too much about RSA!
slug: dont-chat-rsa
ctf: UNR22 Echipe
category: writeup
writeup_categories: ["crypto"]
tags: ["crypto", "RSA"]
summary: Steghide on a picture, AES ECB on the discussion and math to get the p and q of the public key to get the secret.
last_modified_at:
image: https://unr22-individual.cyberedu.ro/images/contests/BMwo9sxqDZaYqNem.png
---

# Legend
* [Challenge Description](#descr)
* [Flag Proof](#flagproof)
* [Summary](#summary)
* [Details](#details)

# Challenge Description {#descr}

Hello Hacker! I've intercepted on the network some interesting stuff: a cute dog photo, two documents with strange random data and someone's public key. I know... I don't know either where to start from... Can you help me make sense of all these files? I think that one of them contains a secret!

Flag format: CTF{message}

# Flag Proof {#flagproof}

> CTF{kn0wiNG_7H3_5um_0f_pRIM35_i5_r3ALLy_DaNg3r0u2}
>

# Summary {#summary}

Steghide on a picture, AES ECB on the discussion and math to get the `p` and `q` of the public key to get the secret.

# Details {#details}

Upon opening the folder we got, we are greeted by an image, two documents and a pubkey. Considering this challenge is also a steganography challenge, I added the image we got into an automatic steg finder.

We got a bunch of results, but we also got a hit on Steghide.

![Untitled](/assets/img/dont-chat-rsa/Untitled.png)

Steghide managed to extract a password `THISwasTOOiziECB`.

After I got this password, I tried to decrypt the conversation in the file named `conversation.txt`. I opened it up in CyberChef, and used it’s AES decryptor.

![Untitled](/assets/img/dont-chat-rsa/Untitled%201.png)

The result was the following conversation, decrypted:

```python
*** internal conversation - level of dissemination - strict confidential ***
[CHRIS]: Hi Steve! It's Chris from the IT Security department. As you've requested, I've changed your old RSA keys to a new set of keys.  
[STEVE]: Awesome! Did you manage to make them secure and fool-proof for me? :)
[CHRIS]: Just secure?!? These keys are super mega ultra extra secure! In fact, they are so secure that nobody will ever break them without a super quantum computer! I don't like brag about it (*typing with a smirk face*), but I choose the prime numbers myself, so they are for sure safe! 
[STEVE]: Cool! Can you tell me the numbers then?
[CHRIS]: Hell NO! It will make your public and private keys worthless! I'll never share those numbers with you.
[STEVE]: Oh, come on maaaan... You know I'm super curious about these things cyber stuff...
[CHRIS]: Sorry, Steve. I cannot tell you what those numbers are. However, what I can tell you is that those numbers are reaaaaallllyyyyyy biiiiiiig and their sum is even bigger... something like 24722116169992465881070271199690169362455458193332180161871370678459930090250287749736220134651803789983780083709278592071250797494670685808152851319806502 ... 
[STEVE]: WOW! That's a huge number indeed!
[CHRIS]: See? I told you that the keys are super safe to use! They are simply UNbreakable! 
[STEVE]: Ok, Chris! Let's see if the keys work now. Send me a secure message and I'll decrypt it. 
*** sending a secure message ***
[STEVE]: Haha, okay, they work. Thanks again Chris! See you at the cafeteria during lunch!
[CHRIS]: No problem! That's my job. See you there!
```

I noticed that they leaked the sum of their two primes.

To get the value n, we use [https://www.dcode.fr/rsa-cipher](https://www.dcode.fr/rsa-cipher) and we paste the public key there.

We get the sum of p+q, and we know p*q=n so in order to get p and q, I created the following script:

```python
import z3
import binascii
from pwn import log, math

n = 152189662152498056403705414170568508936266727797833989074846193193998251501633923591654577149021291177531465603011130507338728477914317649256149828628303152425417194350609637963385945455709522253307371596194326796191907374721561001464612958007488588079796562020731848146769110923462880010550279833363167428057
sum = 24722116169992465881070271199690169362455458193332180161871370678459930090250287749736220134651803789983780083709278592071250797494670685808152851319806502

p, q = z3.Ints('p q')
s = z3.Solver()
s.add(sum == p + q, n == p * q)

assert s.check() == z3.sat, 'Could not find a solution.'

p = s.model()[p].as_long()
q = s.model()[q].as_long()
print(p, q,sep="\n")
```

After getting `p` and `q`, we input the values in [https://www.dcode.fr/rsa-cipher](https://www.dcode.fr/rsa-cipher) again for decrypting the `secret.txt` file (to get an integer from it you convert it to hex and then to decimal), and we get:

`CTF{kn0wiNG_7H3_5um_0f_pRIM35_i5_r3ALLy_DaNg3r0u2}`

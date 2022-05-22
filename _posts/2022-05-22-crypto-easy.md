---
layout: post
author: hofill
title: crypto-easy
slug: crypto-easy
ctf: UNR22 Echipe
category: writeup
writeup_categories: ["crypto"]
tags: ["crypto", "KeePassXC", "AES", "RSA"]
summary: Analyse key for the first flag, then decrypt the second flag using AES CBC and IV.
last_modified_at:
image: https://unr22-individual.cyberedu.ro/images/contests/BMwo9sxqDZaYqNem.png
---

# Legend
* [Challenge Description](#descr)
* [Flag Proof](#flagproof)
* [Summary](#summary)
* [Details](#details)

# Challenge Description {#descr}

You have the safe and a part of the key... Will you be able to find the other part? When opened, pay great attention to contents..

Flag format: CTF{sha256}

# Flag Proof {#flagproof}

First flag:

> CTF{5E6C11B0D5DCB6149BB7205E8966A9F530BC64CFA58DAE1769EA0A6922B9B263}
>

Second flag:

> CTF{1C8EC7DBED7FFF24B7BF5683CD2B818576D7C4EFE1BBD5B79A9FF6BBEED59EF0}
>

# Summary {#summary}

Analyse key for the first flag, then decrypt the second flag using AES CBC and IV.

# Details {#details}

We get two files. A `.key` file and a `.kdbx` file. The `.kdbx` file is used for the application KeePassXC, a password storage option. I can’t make screenshots in the app, so I’ll have to describe what I saw.

I opened up KeePassXC, imported the `.kdbx` file and used the key we were given. We also needed a password, however. I then started analysing the `.key` file. I converted it to hex:

![Untitled](/assets/img/crypto-easy/Untitled.png)

The key in hex looks a lot like the fibonacci sequence, so I tried `fibonacci` as a password, and it worked! This gave me the first flag:

`CTF{5E6C11B0D5DCB6149BB7205E8966A9F530BC64CFA58DAE1769EA0A6922B9B263}`

The second input in the keysafe had a password: `un8r34k4l3`, a note with two hexstrings, a username `aes-256-cbc` and an attachment.

The first thing I thought was decrypting the file with the password and one of the hexstrings as an **IV**. As such, I downloaded the attachment and ran the command:

```python
➜  Downloads openssl enc -d -aes-256-cbc -iv 3235AFF3C541196168C3839B -in flag2.enc -out a.txt 
enter aes-256-cbc decryption password: <wrote password here>
➜  Downloads cat a.txt                                                                         
CTF{1C8EC7DBED7FFF24B7BF5683CD2B818576D7C4EFE1BBD5B79A9FF6BBEED59EF0}%
```

As such, I also got the second flag:

`CTF{1C8EC7DBED7FFF24B7BF5683CD2B818576D7C4EFE1BBD5B79A9FF6BBEED59EF0}`

---
layout: post
author: hofill
title: digital-talking
slug: digital-talking
ctf: UNR22 Individual
category: writeup
writeup_categories: ["crypto", "misc"]
tags: ["crypto", "digital", "7-segment-display", "misc"]
summary: Binary to anode 7 segment display
last_modified_at:
image: https://unr22-individual.cyberedu.ro/images/contests/BMwo9sxqDZaYqNem.png
---

# Legend
* [Challenge Description](#descr)
* [Flag Proof](#flagproof)
* [Summary](#summary)
* [Details](#details)

# Challenge Description {#descr}

Decrypt the message and get the flag.

Don’t forget to put it into the brackets and add CTF.

# Flag Proof {#flagproof}

> CTF{49c2619fc3c6ee789b1fccb46f4ac9af23130293e7cd417b34a97d8160bec5f9}
>

# Summary {#summary}

Binary to anode 7 segment display

# Details {#details}

We get a file with a bunch of binary data. Upon analysing it we realise the following:

- The bytes are grouped by **7**
- There are **17** different ways they are grouped, meaning [0-9a-f], exactly like the flag should look like
- There are **64** of them, enough for the entire flag.

This means that each of the binary chunks represent one letter in the flag.

Taking a hint from the title of the challenge, we realise that we are talking about something `digital`. One of the things that came to mind was the *7 segment display.*

I then used [https://www.dcode.fr/7-segment-display](https://www.dcode.fr/7-segment-display) to decode the binary data, and I got:

![image](/assets/img/digital-talking/1.png)

I made the text lowercase and submitted the flag.

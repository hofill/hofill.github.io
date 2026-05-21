---
layout: post
author: hofill
title: "AES CTF Tool — Fingerprinting and Attacking Block Ciphers"
slug: aes-ctf-tool
category: blog
categories: blog
tags: ["crypto", "aes", "tools", "python", "ctf"]
summary: My diploma thesis turned into a Python tool that fingerprints AES block cipher modes from ciphertext alone and automatically launches attacks.
last_modified_at:
---

For my diploma thesis at Babeș-Bolyai University I ended up building something I actually wanted to use in CTFs — a Python tool that looks at a ciphertext (and optionally a server), figures out which AES mode of operation is being used, and then automatically exploits it.

The paper is called *Cryptanalysis and Fingerprinting of Block Cipher Modes of Operation*. The code lives [on GitHub](https://github.com/hofill).

## The problem

When you're in a CTF and hit a crypto challenge with an AES oracle, the first thing you need to know is which mode is being used — ECB, CBC, CFB, OFB, or CTR. Most of the time you're doing this manually by hand, encrypting a few inputs and eyeballing the output lengths and patterns. This gets tedious fast, especially if the attack steps that follow (Padding Oracle, ECB Chosen Plaintext) are also manual.

The idea was to automate the whole pipeline: connect to the server, fingerprint the mode, then launch the appropriate attack.

## Fingerprinting from ciphertext alone

Even without access to the plaintext, a ciphertext leaks a lot about the mode used.

**Length tells you a lot.** ECB and CBC require padding to align to 16-byte blocks. CFB, OFB, and CTR behave like stream ciphers and don't pad — the ciphertext is exactly the same length as the plaintext. So if the ciphertext isn't a multiple of 16 bytes, you're almost certainly looking at CFB, OFB, or CTR.

**Identical blocks = ECB.** ECB encrypts each 16-byte block independently with the same key, so identical plaintext blocks always produce identical ciphertext blocks. If you see repeated 16-byte chunks in the ciphertext, it's a very strong signal for ECB. How strong? The probability of two random blocks colliding by chance is:

```
P(collision) = 1 - (1 × (1 - 1/256^16) × ...) ≈ 2.938 × 10^-39
```

So basically impossible — if you see it, it's ECB.

**IV length distinguishes stream-like modes.** CTR uses a nonce that's shorter than the IV used by CFB and OFB. That size difference shows up in the ciphertext when the IV/nonce is prepended.

## Fingerprinting with a plaintext oracle

If you can encrypt chosen plaintexts, you get more precision:

**Block size detection.** Encrypt progressively longer strings (`a`, `aa`, `aaa`, ...) until the ciphertext length jumps. The jump tells you the block size.

**Padding method.** Send exactly `block_size` bytes, then `block_size + 1` bytes. If the first produces an extra block, the server uses PKCS#7 (which always appends padding). You can also detect the PKCS#7 signature by encrypting `0x10 * 16` and checking if the result contains two identical blocks — that's the padding block giving itself away.

**ECB vs CBC.** Once you know the block size, encrypt `block_size * 3` identical bytes. If you get two identical ciphertext blocks in the middle, it's ECB. CBC would chain the XOR from the previous ciphertext block and break the pattern.

## The attacks

Once the mode is identified, the tool automatically prompts you to launch the relevant attack.

### ECB — Chosen Plaintext Attack

ECB's lack of diffusion means you can recover an appended secret byte by byte. The approach:

1. Find the offset by prepending bytes until two consecutive ciphertext blocks become identical.
2. Encrypt `offset + (block_size - 1)` known bytes. The target byte ends up as the last byte of a block you control.
3. Bruteforce that last byte by trying all 256 values and comparing ciphertext blocks.
4. Repeat for each byte of the secret.

### CBC — Padding Oracle Attack

If a server tells you whether a decrypted ciphertext has valid PKCS#7 padding (even just via a different error message), you can recover the full plaintext without knowing the key.

The attack works on the "intermediate state" — the output of the block cipher before the XOR with the previous ciphertext block. By flipping bytes in a crafted ciphertext block and observing when the padding validates, you recover the intermediate state and from it the plaintext.

## The code

The codebase is built around a `BCDetector` base class. To use it against a real server, you subclass it and implement three methods:

```python
class Det(BCDetector):
    def init_server(self):
        return process(["./test_servers/ecb.py"])

    def encrypt(self, data, server):
        server.recvuntil(b">")
        server.sendline(b"1")
        server.sendline(data.encode())
        return server.readline().strip().split(b": ")[1].decode()

    def decrypt(self, data, server):
        server.recvuntil(b">")
        server.sendline(b"2")
        server.sendline(data.encode())
        response = server.readline().strip()
        if response == b"Invalid padding":
            raise BadPaddingException
        return response.split(b": ")[1].decode()

if __name__ == "__main__":
    detector = Det()
    detector.begin()
```

`begin()` handles everything: fingerprinting, probability reporting, and attack prompting.

The output looks like this when pointed at an ECB server:

```
[INFO] Starting initial cryptanalysis.
[INFO] Determining block size.
[X] Found block size: 16.
[INFO] Determining block cipher category.
[X] Found block cipher category: ECB_CBC.
[INFO] Starting fingerprinting.
[INFO] Determining block cipher mode.
[X] Found block cipher mode: ECB.
======= Probabilities =======
ECB: 100%
CBC:   0%
CFB:   0%
OFB:   0%
CTR:   0%
=============================
[INFO] ECB/CBC detected. Determining padding method.
[X] Found padding method: PKCS7.
[INFO] Fingerprinting complete.
```

Then it asks if you want to launch the Chosen Plaintext Attack.

The `BCState` class manages the probabilistic state across multiple plaintext-ciphertext pairs, and the `Certainty` class tracks confidence levels per mode. The design is intentionally modular — any server that can encrypt (and optionally decrypt) can be plugged in with minimal boilerplate.

## Why I built this

I kept doing the same analysis steps manually in every CTF crypto challenge. The fingerprinting part especially — counting bytes, checking for repeated blocks, calculating block sizes — is mechanical enough that it should be automated. The attacks themselves are also well-documented enough that the implementation should be reusable.

The tool ended up being the practical output of the thesis, which was otherwise a formal study of the differences between AES modes and their attack surfaces. Hopefully it's useful to others grinding crypto CTF challenges too.

The full paper and code are both available on [my GitHub](https://github.com/hofill).

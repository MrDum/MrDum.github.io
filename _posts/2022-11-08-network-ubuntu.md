---
title:  "Ubuntu netplan"
header:
  teaser: "/assets/images/kevin-ku-w7ZyuGYNpRQ-unsplash.jpg"
tags:
  - ubuntu
---

# Przykładowa konfiguracja sieci ubuntu netplan

```python
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      addresses:
        - 192.168.0.21/24
      nameservers:
        search: [local, local.reszt.xyz]
        addresses: [192.168.0.20, 1.1.1.1]
      routes:
        - to: default
          via: 192.168.0.1
```
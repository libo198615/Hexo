---
title: react-native-error
date: 2019-04-01 16:19:19
categories:
- react-native
tags:
---

```javascript
mac listen EADDRINUSE: address already in use :::8081
端口被占用

第一步：sudo lsof -i:8081

➜  Bill sudo lsof -i:8081
COMMAND   PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
node    13654 libo   23u  IPv6 0xc9988e691127882f      0t0  TCP *:sunproxyadmin (LISTEN)
node    13654 libo   36u  IPv6 0xc9988e6911f0efef      0t0  TCP localhost:sunproxyadmin->localhost:58843 (ESTABLISHED)
Bill    23847 libo   10u  IPv6 0xc9988e6911f0fb6f      0t0  TCP localhost:58843->localhost:sunproxyadmin (ESTABLISHED)
Bill    23847 libo   11u  IPv6 0xc9988e6911f0fb6f      0t0  TCP localhost:58843->localhost:sunproxyadmin (ESTABLISHED)

第二步：sudo kill 13654
```


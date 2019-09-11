---
title: RSA加密
date: 2019-07-05 10:52:55
tags:
---





查看.cer
```shell
openssl x509 -in CERT.cer -text
```


查看 .pem
```shell
cat private.pem
```



pfx  文件转  pem 文件
```shell
openssl pkcs12 -in /Users/admin/Desktop/private.pfx -nodes -out server.pem
```


从私钥中获取公钥 Private.pem -> public.pem
```shell
openssl rsa -in /Users/admin/Downloads/key/private.pem  -pubout -out public.pem
```


查看 .pfx 
```shell
// END CERTIFICATE
openssl pkcs12 -in /Users/admin/Desktop/private.pfx  -clcerts 

// BEGIN PRIVATE KEY
openssl pkcs12 -in /Users/admin/Desktop/private.pfx  -nocerts -nodes 
```


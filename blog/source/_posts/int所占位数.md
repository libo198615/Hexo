---
title: int long 所占位数
date: 2019-01-03 17:22:10
categories:
- iOS
tags:
---



重点注意 long
// 32 
`int64_t` `double` `long long` 8位，其他4位
// 64
`int` `int32_t` 4位，其他8位


```
//    // 32                           // 位数  数值
//    int         t1 = pow(2, 31);    // 4    2147483647
//    int32_t     t2 = pow(2, 31);    // 4    2147483647
//    int64_t     t3 = pow(2, 62);    // 8    4611686018427387904
//    NSInteger   t4 = pow(2, 32);    // 4    2147483647
//    double      t5 = pow(2, 64);    // 8    18446744073709551616.000000
//    long        t6 = pow(2, 31);    // 4    2147483647
//    long long   t7 = pow(2, 62);    // 8    4611686018427387904

//    // 64                           // 位数  数值
//    int         t1 = pow(2, 31);    // 4    2147483647
//    int32_t     t2 = pow(2, 31);    // 4    2147483647
//    int64_t     t3 = pow(2, 63);    // 8    9223372036854775807
//    NSInteger   t4 = pow(2, 63);    // 8    9223372036854775807
//    double      t5 = pow(2, 64);    // 8    18446744073709551616.000000
//    long        t6 = pow(2, 63);    // 8    9223372036854775807
//    long long   t7 = pow(2, 63);    // 8    9223372036854775807
```

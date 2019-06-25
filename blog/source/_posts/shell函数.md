---
title: shell函数
date: 2019-01-06 23:18:09
categories:
- SHELL命令
tags:
---


##### 传递参数

$0 为执行的文件名
```
echo "执行的文件名：$0";
echo "第一个参数为：$1";
```
```
$ chmod +x test.sh 
$ ./test.sh 1 
```
```
执行的文件名：./test.sh
第一个参数为：1
```


##### 函数
```
demoFun()
{
    echo "这是我的第一个 shell 函数!"
}

echo "-----函数开始执行-----"
demoFun
echo "-----函数执行完毕-----"

#-----函数开始执行-----
#这是我的第一个 shell 函数!
#-----函数执行完毕-----
```
返回值通过 $？ 获取
```
funWithReturn(){
    echo "这个函数会对输入的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
echo "输入的两个数字之和为 $? !"

// 输出
这个函数会对输入的两个数字进行相加运算...
输入第一个数字: 
1
输入第二个数字: 
2
两个数字分别为 1 和 2 !
输入的两个数字之和为 3 !
```
注意，10不能获取第十个参数，获取第十个参数需要{10}。当n>=10时，需要使用${n}来获取参数。
```
funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73// 输出
第一个参数为 1 !
第二个参数为 2 !
第十个参数为 10 !
第十个参数为 34 !
第十一个参数为 73 !
参数总数有 11 个!
作为一个字符串输出所有参数 1 2 3 4 5 6 7 8 9 34 73 !
```

##### 字符串

单引号
```
str='this is a string'
```
单引号字符串的限制：
* 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的
* 单引号字串中不能出现单引号（对单引号使用转义符后也不行）
双引号
```
your_name='qinjx'
str="Hello, I know your are \"$your_name\"! \n"
```
* 双引号里可以有变量
* 双引号里可以出现转义字符

##### 数组

- 创建数组 用空格分开
```
arr=(A B "C" D)
```
- 读取数组
```
echo "${arr[0]}"
```
- 获取数组中的所有元素
```
echo "${arr[*]}" // A B C D
echo "${arr[@]}" // A B C D
```
- 获取数组长度
```
echo "${#arr[*]}" // 4
echo "${#arr[@]}" // 4
```

##### 流程控制

sh的流程控制不可为空，如果else分支没有语句执行，就不要写这个else
```
if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```
以下实例判断两个变量是否相等：
```
a=10
b=20
if [ $a == $b ]
then
    echo "a 等于 b"
elif [ $a -gt $b ]
then
    echo "a 大于 b"
elif [ $a -lt $b ]
then
    echo "a 小于 b"
else
    echo "没有符合的条件"
fi
```
- for 循环
```
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```
输出结果：
```
The value is: 1
The value is: 2
The value is: 3
The value is: 4
The value is: 5
```


- while 语句

while循环用于不断执行一系列命令，也用于从输入文件中读取数据；命令通常为测试条件。其格式为：
```
while condition
do
command
done
```


- 跳出循环

在循环过程中，有时候需要在未达到循环结束条件时强制跳出循环，Shell使用两个命令来实现该功能：break和continue。

break命令

break命令允许跳出所有循环（终止执行后面的所有循环）

continue

continue仅仅跳出当前循环。



##### 点滴记录

- 当前文件目录
```
outPath=`dirname $0`/OUT/
echo $outPath

#/Users/megvii/Desktop/test/OUT/
```
#dirname用于取给定路径的目录部分,$0是Shell本身的文件名。

- 16进制内容
```
xxd  /Users/megvii/Downloads/MegviiFaceppv2_iOS_0.4.7/facepp_iOS/model/megviifacep
```
- 显示文件哈希值
```
shasum  /Users/megvii/model/megviifacepp_0_4_7_model
```
- 显示文件 MD5
```
openssl md5 /Users/megvii/model/megviifacepp_0_4_7_model
```
安装过程
```
./configure // 生成 Makefile
make  // 编译
make install // 安装

$ cat ~/.ssh/id_rsa.pub

// 显示文件内容 
// 看是否有自己写的注释
strings  /../../file

file // 确定文件类型

// arm64 armv7
lipo -info   xx.a
```




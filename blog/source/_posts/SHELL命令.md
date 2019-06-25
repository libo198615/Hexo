---
title: SHELL命令
date: 2019-01-06 23:04:37
categories:
- SHELL命令
tags:
---

##### mkdir

选项与参数：

* -m ：配置文件的权限喔！直接配置，不需要看默认权限 (umask) 的脸色～
* -p ：帮助你直接将所需要的目录(包含上一级目录)递归创建起来！

在目录/usr/meng下建立子目录test，并且只有文件主有读、写和执行权限，其他人无权访问 
```
mkdir -m 700 /usr/meng/test 
```
在当前目录中建立bin和bin下的os_1目录，权限设置为文件主可读、写、执行，同组用户可读和执行，其他用户无权访问
```
mkdir -p-m 750 bin/os_1
```

##### rm
```
// 删除 models文件夹 和里面所有内容
rm -rf models
```

选项与参数：

* -f ：就是 force 的意思，忽略不存在的文件，不会出现警告信息；
* -i ：互动模式，在删除前会询问使用者是否动作
* -r ：递归删除啊！最常用在目录的删除了！这是非常危险的选项！！！

将刚刚在 cp 的范例中创建的 bashrc 删除掉！
```
rm -i bashrc
rm: remove regular file `bashrc'? y
```
如果加上 -i 的选项就会主动询问喔，避免你删除到错误的档名！

##### rmdir

删除空文件夹
```
rmdir -p aa/bb // 如果删除bb后aa为空目录，则将aa一并删除
```
```
if [ -d "MGFacepp/build" ]; then
    rm -rf MGFacepp/build
fi
```



##### cd
```
// 调到当前目录的上上两层
cd ../..
```
`cd` (切换目录)

`cd`是`Change Directory`的缩写，这是用来变换工作目录的命令。

##### cp

文件夹重名要先删除已经存在的文件夹  jenkins 和 终端 有区别
```
cp -r test/ newTest
YES|cp -r test/ newTest 有重名文件则覆盖
```
选项与参数：

* **-i：**若目标档(destination)已经存在时，在覆盖时会先询问动作的进行(常用)
* **-p：**连同文件的属性一起复制过去，而非使用默认属性(备份常用)；
* **-r：**递归持续复制，用於目录的复制行为；(常用)

##### ls
```
ls -a // 显示所有档案及目录
```

选项与参数：

* -a ：全部的文件，连同隐藏档
* -l ：长数据串列出，包含文件的属性与权限等等数据；

##### echo

- 显示普通字符串
```
echo "name" // name
echo name   // name
```
- 显示转义字符
```
echo "\"name\"" // "name"
```
- 显示变量
```
read name // 从命令行读取用户输入
echo "your name is $name"
```
- 显示换行
```
echo -e "OK! \n" # -e 开启转义
echo hello

OK!

hello
```

- 显示时间
```
echo `date`
```


##### grep

* -H或--with-filename 在显示符合范本样式的那一列之前，表示该列所属的文件名称。
* -l或--file-with-matches 列出文件内容符合指定的范本样式的文件名称。
* -L或--files-without-match 列出文件内容不符合指定的范本样式的文件名称。
* -n或--line-number 在显示符合范本样式的那一列之前，标示出该列的列数编号。
```
grep -Hn  'print'  pack_all.py

pack_all.py:36:    print ("########### read modle info ############")
pack_all.py:38:        print(line)
pack_all.py:60:    print ("model info = %s" % model_info)
pack_all.py:78:    print (confsList)
pack_all.py:80:    print ("confsList = %s" % (confsList))
pack_all.py:82:    print ("confs_real = %s" % confs_real)
pack_all.py:88:    print (config_offsets)
pack_all.py:94:        print ("confs_real.len = %s" % len(confs_real))
pack_all.py:102:        print ("json_str length = %s" % len(json_str))
pack_all.py:111:        print ("config_size_map = %s" % config_size_map)
pack_all.py:113:#            print(x, config_size_map[x], model_dict[real2fake[x]
```

##### lipo

- 查看静态库支持的CPU架构
```
lipo -info file.a
```
- 合并静态度
```
lipo -create 静态库路径1  静态库路径2   -output 整合后的静态库路径

lipo -create file1.a file2.a  -output  newFile.a
```
- 静态库拆分
```
lipo file.a  -thin  armv7  -output armv7.a
```


##### ln 软链接
```
cd 到要软链接到的目录
ln -s ../../../要链接的文件
// 按两下 tab 键可以显示../../ 目录下文件
```
```
ln -s // symbolic link
ln  // hard link
```
文件分为用户数据 (user data) 与元数据 (metadata)。
用户数据，即文件数据块 (data block)，数据块是记录文件真实内容的地方；
而元数据则是文件的附加属性，如文件大小、创建时间、所有者等信息。在 Linux 中，元数据中的 inode 号（inode 是文件元数据的一部分但其并不包含文件名，inode 号即索引节点号）才是文件的唯一标识而非文件名。文件名仅是为了方便人们的记忆和使用，系统或程序通过 inode 号寻找正确的文件数据块

硬链接或软链接都不会将原始文档复制一份
软连接以路径的形式存在，可以对目录进行链接，
硬链接以文件副本的形式存在，但不占用实际空间，删除原始文件后，硬链接不受影响
硬链接、软链接有不同的元数据。 软链接有不同的inode号，硬链接有相同的inode号，所以硬链接限制更多，不能跨操作系统。

```
// 拖拽多个文件
ln -s 拖拽的多个文件 目标文件夹  
```


##### pwd

print working directory 打印当前工作目录
```
pwd
/Users/megvii/....
```
```
// 将当前目录压入堆栈
pushd `pwd`
```


- pushd：

* 切换到作为参数的目录，并把原目录和当前目录压入到一个虚拟的堆栈中

* 如果不指定参数，则会回到前一个目录，并把堆栈中最近的两个目录作交换

- popd： 

* 切换到堆栈中第二目录中，并将堆栈中最近的目录弹出

- dirs: 

* 列出当前堆栈中保存的目录列表

-v参数可以在目录前加上编号
我们可以看到:最近压入堆栈的目录位于最上面
在最近的两个目录之间切换:用pushd不加参数即可
```
pushd /boot/grub/
/boot/grub /usr/share/kde4/apps/kget /usr/local/sbin ~
dirs -v
0  /boot/grub
1  /usr/share/kde4/apps/kget
2  /usr/local/sbin
3  ~

pushd
/usr/share/kde4/apps/kget /boot/grub /usr/local/sbin ~
dirs -v
0  /usr/share/kde4/apps/kget
1  /boot/grub
2  /usr/local/sbin
3  ~

pushd
/boot/grub /usr/share/kde4/apps/kget /usr/local/sbin ~
dirs -v
0  /boot/grub
1  /usr/share/kde4/apps/kget
2  /usr/local/sbin
3  ~

如何把目录从堆栈中删除?用popd即可

dirs -v
0  /usr/local/sbin
1  ~
2  /boot/grub
3  /usr/share/kde4/apps/kget

popd
~ /boot/grub /usr/share/kde4/apps/kget
dirs -v
0  ~
1  /boot/grub
2  /usr/share/kde4/apps/kget

popd +1
~ /usr/share/kde4/apps/kget
dirs -v
0  ~
1  /usr/share/kde4/apps/kget
```

说明：可以看到popd不加参数的运行情况:popd把堆栈顶端的目录从堆栈中删除，并切换于位于新的顶端的目录
说明之二: popd 加有参数 +n时，n是堆栈中的第n个目录，表示把堆栈中第n个目录从堆栈中删除
dirs可以清空目录堆栈 用 -c参数即可
```
dirs -v
0  ~
1  /boot/grub
2  /usr/share/kde4/apps/kget
dirs -c
dirs -v
0  ~
```


##### strings 
```
// 是否包含 aaa 不区分大小写
strings file | grep -i aaa
```

##### sudo

以系统管理者的身份执行命令


##### tree

以树状图列出目录内容
```
tree -C // 在文件和目录清单上加上色彩，便于区分各种类型
```
```
tree -L 2
```

##### vim

- 插入模式
```
i 进入
esc 退出
```
```
// 终端进入vim
vim  进入
1. esc  
2. :wq(保存退出) / :q!(不保存退出)
```

- 创建文件
```
// 终端下
vim test.txt
```
```
chmod +x ./test.sh  #使脚本具有执行权限
./test.sh  #执行脚本
```

##### zip  tar
```
// 将 /home/html/目录下的所有文件夹和文件打包为当前目录下的 html.zip
zip -q -r html.zip /home/html
// -q 不显示指令执行过程
// -r 递归处理
```
```
// 压缩 a.c 为 test.tar
tar -czvf test.tar a.c
// -c (--create)建立新的备份文件
// -z (--gzip)(--ungzip)通过gzip指令处理备份文件
// -v (--verbose)显示指令执行过程
// -f (--file)指定备份文件
```
```
// 解压文件
tar -xzvf test.tar a.c
// -x (--extract)(-get)从备份文件中还原文件
```





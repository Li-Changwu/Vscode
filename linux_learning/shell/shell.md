# Shell学习

[TOC]

## 1.Shell 概述

Shell是一个**命令行解释器**，它接收应用程序/用户命令，然后调用操作系统内核。还是一个功能相当强大的**编程语言**，易编写、易调试、灵活性强。
![shell](/assets/shell.png)
查看内置shell解析器，centOS默认使用的bash,通过软连接将sh指向bash

```shell
[root@chunis ~]# cat /etc/shells 
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
/bin/tcsh
/bin/csh
```

## 2.shell脚本入门

### 脚本格式

脚本以#!/bin/bash 开头（指定解析器），文件默认以sh为后缀。如 hello.sh

```shell
#!/bin/bash 
echo "helloworld"
```

### 脚本执行方式

1. 采用 bash 或 sh+脚本的相对路径或绝对路径（不用赋予脚本+x 权限）
  
    ```shell
    sh ./helloworld.sh # 相对路径
    sh /home/shells/helloworld.sh # 绝对路径
    bash ./helloworld.sh # 相对路径
    bash /home/shells/helloworld.sh # 绝对路径
    ```

2. 采用输入脚本的绝对路径或相对路径执行脚本（必须具有可执行权限+x）

    ```shell
    chmod +x helloworld.sh # 要赋予 helloworld.sh 脚本的执行权限
    ./helloworld.sh # 相对路径执行脚本
    /home/shells/helloworld.sh # 绝对路径执行脚本
    ```

3. 在脚本的路径前加上“.”或者 source

    ```shell
    . hello.sh
    source hello.sh
    ```

**注意**：第一种执行方法，本质是 bash 解析器帮你执行脚本，所以脚本本身不需要执行权限。第二种执行方法，本质是脚本需要自己执行，所以需要执行权限。前两种方式都是在当前shell中打开一个子shell来执行脚本内容,当脚本内容结束，则子shell关闭，回到父shell中。
第三种，也就是使用在脚本路径前加“.”或者 source 的方式，可以使脚本内容在当前 shell 里执行，而无需打开子 shell！这也是为什么我们每次要修改完/etc/profile 文件以后，需要source一下的原因。

**开子shell与不开子shell的区别**在于环境变量的继承关系，如在子shell中设置的 当前变量，父 shell 是不可见的

## 3.变量

### 系统常用变量

```shell
$HOME、$PWD、$SHELL、$USER等
```

### 查看变量

查看系统变量的值,如:```echo \$HOME```

显示当前 Shell 中的环境(全局)变量：```env```

打印当前 Shell 中的某个环境(全局)变量：```printenv HOME、echo $HOME```

显示当前 Shell 中所有变量：set

### 自定义变量

定义变量：变量名=变量值。**注意：**=号前后不能有空格

撤销变量：unset 变量名

声明只读(静态)变量：readonly 变量名=变量值,如：```readonly a=1```，注意：不能 unset

提升局部变量为全局变量：```export 变量名```,如：```export a```

变量默认类型都是字符串类型，无法直接进行数值运算,变量的值如果有空格，需要使用双引号或单引号括起来

### 特殊变量

+ ```$n```
功能描述：n 为数字，$0 代表该脚本名称，$1-$9 代表第一到第九个参数，十以 上的参数，十以上的参数需要用大括号包含，如${10}

+ ```$#```
功能描述：获取所有输入参数个数，常用于循环,判断参数的个数是否正确以及 加强脚本的健壮性

+ ```$*、$@```
功能描述：变量代表命令行中所有的参数，**```$*```把所有的参数看成一个整体，不过```$@```把每个参数区分对待**

+ ```$?```
功能描述：最后一次执行的命令的返回状态。如果这个变量的值为 0，证明上一 个命令正确执行；如果这个变量的值为非 0（具体是哪个数，由命令自己来决定），则证明 上一个命令执行不正确了

    **实例：**

    ```shell
    [root@chunis scripts]# cat var.sh
    #!/bin/bash
    echo '==========$n=========='
    # 命令替换：使用“$(系统命令)”调用系统命令
    echo script name:$(basename $0 .sh)
    echo scipt path:$(dirname $0) 
    echo $1
    echo $2
    echo '==========$#=========='
    echo $#
    echo '==========$*=========='
    echo $*
    echo '==========$@=========='
    echo $@
    [root@chunis scripts]# ./var.sh a b c d e f g
    ==========$n==========
    script name:var
    scipt path:.
    a
    b
    ==========$#==========
    7
    ==========$*==========
    a b c d e f g
    ==========$@==========
    a b c d e f g
    [root@chunis scripts]# echo $?
    0
    ```

## 4.运算符

基本语法：```$((运算式)) 或 $[运算式]```

实例：

```shell
[root@hadoop100 ~]# clear
[root@hadoop100 ~]# a=$[2+3*4]
[root@hadoop100 ~]# echo $a
14
[root@hadoop100 ~]# b=$((2-3*4))
[root@hadoop100 ~]# echo $b
-10
```

## 5.条件判断

基本语法:

1. ```test condition```
2. ```[ condition ]```(常用)
   **注意**:condition 前后要有空格；条件非空即为true，[ atguigu ]返回true，[ ] 返回fals

常用判断条件:

1. 两个整数之间比较
-eq 等于（equal）  -ne 不等于（not equal）
-lt 小于（less than） -le 小于等于（less equal）
-gt 大于（greater than） -ge 大于等于（greater equal）
**注**：如果是字符串之间的比较 ，用等号“=”判断相等；用“!=”判断不等。

    ```shell
    [ 23 -eq 22 ]
    ```

2. 按照文件权限进行判断
-r 有读的权限（read）
-w 有写的权限（write）
-x 有执行的权限（execute）

    ```shell
    [ -x hello.sh ] # 查看是否有执行权限
    echo $? # 查看返回结果
    ```

3. 按照文件类型进行判断
-e 文件存在（existence）
-f 文件存在并且是一个常规的文件（file）
-d 文件存在并且是一个目录（directory

    ```shell
    [ -e /root/scripts/hello.sh ] # 查看文件是否存在
    echo $? # 查看返回结果
    ```

    使用逻辑与&&加逻辑||实现三目运算符

    ```shell
    [ $[ 1 + 1 ] -eq 2 ] && echo equal || echo notequal
    ```

## 6.流程控制（重点）

### if判断

```shell
# 1.[ 条件判断式 ]，中括号和条件判断式之间必须有空格
# 2. if 后要有空格
if [ 条件判断式 ]
then
程序
elif [ 条件判断式 ]
then
程序
else
程序
fi
```

判断条件为多个时：

```shell
# -a 表示逻辑与&& -o 表示逻辑或||
if [ condition1 -a conditon2 ]
# 或者
if [ condition1 ] && [ conditon2 ]
```

### case语句

基本语法：

```shell
case $变量名 in
"值1"）
如果变量的值等于值1，则执行程序1
;;
"值2"）
如果变量的值等于值2，则执行程序2
;;
...省略其他分支...

*）
如果变量的值都不是以上的值，则执行此程序
;;
esac
```

注意事项：
（1）case 行尾必须为单词“in”，每一个模式匹配必须以右括号“）”结束。
（2）双分号“;;”表示命令序列结束，相当于java 中的break。
（3）最后的“*）”表示默认模式，相当于java 中的default

### for循环

基本语法1:

```shell
for (( 初始值;循环控制条件;变量变化 ))
do
程序
done
```

实例：

```shell
[root@hadoop100 scripts]# cat for1.sh 
#!/bin/bash

sum=0
for (( i=1; i<=$1; i++ ))
do
  sum=$[$sum+$i]
done
echo $sum
[root@hadoop100 scripts]# chmod +x for1.sh 
[root@hadoop100 scripts]# ./for1.sh 10
45
```

基本语法2

```shell
for 变量 in 值1 值2 值3...
do
程序
done
```

实例1：

```shell
[root@hadoop100 scripts]# cat for3.sh 
#!/bin/bash

# {}表示序列，{1..5}表示生成一个1到5的序列
for i in {1..5}
do
  echo $i
done
[root@hadoop100 scripts]# ./for3.sh 
1
2
3
4
5
```

实例2：

```shell
[root@hadoop100 scripts]# cat for2.sh 
#!/bin/bash

echo '========$@=========='
for i in $@
do
  echo $i
done

echo '========$*========='
for i in $*
do
  echo $i
done

echo '========"$@"=========='
for i in "$@"
do
  echo $i
done

echo '========"$*"=========='
for i in "$*"
do
  echo $i
done
[root@hadoop100 scripts]# chmod +x for2.sh 
[root@hadoop100 scripts]# ./for2.sh t1 t2 t3
========$@==========
t1
t2
t3
========$*=========
t1
t2
t3
========"$@"==========
t1
t2
t3
========"$*"==========
t1 t2 t3
```

\$*和\$@都表示传递给函数或脚本的所有参数，不被双引号“”包含时，都以\$1 \$2 ...\$n的形式输出所有参数

当它们被双引号“”包含时，\$*会将所有的参数作为一个整体，以“\$1 \$2 ...\$n”的形式输出所有参数；$@会将各个参数分开，以“\$1” “\$2”...“\$n”的形式输出所有参数。

### while循环

基本语法：

```shell
while [ 条件判断式 ]
do
    程序
done
```

实例：

```shell
[root@hadoop100 scripts]# cat while.sh 
#!/bin/bash
#calculate the sum from 1 to n

sum=0
i=1
while [ $i -le $1 ]
do
  sum=$[$sum+$i]
  i=$[$i+1]
done
echo $sum

[root@hadoop100 scripts]# chmod +x while.sh 
[root@hadoop100 scripts]# ./while.sh 5
15
```

## 7.read 读取控制台输入

基本语法:read (选项) (参数)
选项：
-p：指定读取值时的提示符；
-t：指定读取值时等待的时间（秒）如果-t 不加表示一直等待
参数:变量（指定读取值的变量名）

```shell
[root@hadoop100 scripts]# cat read.sh 
#!/bin/sh

read -t 7 -p "Enter your name in 7 seconds:" name
echo hello,$name
[root@hadoop100 scripts]# chmod +x read.sh 
[root@hadoop100 scripts]# ./read.sh
Enter your name in 7 seconds:lcw
hello,lcw
```

## 8.函数

### 系统函数

1. basename
基本语法：basename [string / pathname] [suffix]
功能描述：basename 命令会删掉所有的前缀包括最后一个（‘/’）字符，然后将字符串显示出来。basename 可以理解为取路径里的文件名称
选项：
suffix 为后缀，如果suffix 被指定了，basename 会将pathname 或string 中的suffix 去掉

    ```shell
    [root@hadoop100 scripts]# basename ./while.sh .sh
    while
    ```

2. dirname
基本语法：dirname 文件路径
功能描述：从给定的包含路径的文件名中去除文件名
（非目录的部分），然后返回剩下的路径（目录的部分），dirname 可以理解为取文件路径的路径名称

    ```shell
    [root@hadoop100 scripts]# dirname ./while.sh 
    .
    [root@hadoop100 scripts]# dirname /root/scripts/while.sh 
    /root/scripts
    ```

### 自定义函数

基本语法：

```shell
# 这里[]表示可以不写
[ function ] funname[()]
{
    程序
    [return int;]
}
```

**注意**：
（1）必须在调用函数地方之前，先声明函数，shell 脚本是逐行运行，不会像其它语言一样先编译。
（2）函数返回值，只能通过$?系统变量获得，可以显示加return 返回，如果不加，将以最后一条命令运行结果，作为返回值，返回值范围为(0-255)。

实例：

```shell
[root@hadoop100 scripts]# cat ./func.sh 
#!/bin/bash

function sum()
{
  sum=$[$1+$2]
  echo $sum
}

read -p "请输入数字1:" num1
read -p "请输入数字2:" num2

sum=$(sum num1 num2)
echo "$num1 + $num2 = $sum"
[root@hadoop100 scripts]# ./func.sh
请输入数字1:11
请输入数字2:456
11 + 456 = 467
```

## 9.正则表达式入门

## 10.文本处理工具

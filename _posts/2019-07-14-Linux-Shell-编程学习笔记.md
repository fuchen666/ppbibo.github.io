---
layout: post
title: "Linux Shell 编程学习笔记"
subtitle: "Linux Shell 编程学习笔记"
date: 2019-07-14
author: ppbibo
category: shell
finished: true
---
[TOC]

#### PS：

之前写过一个Linux 基线脚本用于centos7 系统加固（仿照青藤云的）shell 命令还是很方便的，对于经常在Linux系统下工作的是非常有帮助的，所以在好好看一遍，记个笔记，脑子笨就多写写。



## 0x1 shell 变量

#### 1. 普通变量

```bash
Mac-PpBiBo $ a='hello shell'
Mac-PpBiBo $ b=123
Mac-PpBiBo $ echo $a,$b
hello shell,123
Mac-PpBiBo $ 

# 注意定义变量的时候不要有空格
```

#### 2. 只读变量

```bash
Mac-PpBiBo $ c="read string"
Mac-PpBiBo $ re
Mac-PpBiBo $ readonly c
Mac-PpBiBo $ echo $c
read string
Mac-PpBiBo $ c="asdasd"
-bash: c: readonly variable
Mac-PpBiBo $ 

# 只读变量顾名思义只允许读取的变量，如上重新赋值会报错。（ -bash: c: readonly variable ）
```

#### 3. 使用变量

```bash
echo {$a} 或者 echo $a
c=$a

PS:只有在使用变量时，变量名前需要加$符号,{}可选当然最好使用。
```

#### 4. 销毁变量

```bash
unset a   # 只读变量是没法删除的
```

#### 5. 特殊变量

```bash
$0	当前脚本的文件名
$n	传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。
$#	传递给脚本或函数的参数个数。
$*	传递给脚本或函数的所有参数。
$@	传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。
$?	上个命令的退出状态，或函数的返回值。
$$	当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。
```



## 0x2 shell数据结构

#### 1. 字符串

```bash
Mac-PpBiBo $ str='this string'
Mac-PpBiBo $ echo $str
this string

```

**Tag：**

*1.单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的*

*2.单引号字符串中不能出现单引号（对单引号使用转义符后也不行）*

*3.双引号里可以有变量*

*4.双引号里可以出现转义字符*

#### 2. 字符串与字符串变量的拼接

```bash
Mac-PpBiBo $ str01="str01 print $str"
Mac-PpBiBo $ echo $str01
str01 print this string
Mac-PpBiBo $ str02="this is {$str01}"
Mac-PpBiBo $ echo $str02
this is {str01 print this string}
Mac-PpBiBo $ str03="this is "$str02""
Mac-PpBiBo $ echo $str03
this is this is {str01 print this string}
Mac-PpBiBo $ 

```

#### 3. 获取字符串长度

```bash
Mac-PpBiBo $ echo $str03
this is this is {str01 print this string}
Mac-PpBiBo $ echo ${#str03}
41

```

#### 4. 字符串切片

```bash
Mac-PpBiBo $ string="this is a test"
Mac-PpBiBo $ echo ${string:9:13} # 输出test
Mac-PpBiBo $ echo ${string:9}
test

```

#### 4.1 "#" 号截取，删除左边字符，保留右边字符。

```bash
Mac-PpBiBo $ var="http://www.baidu.com/index.html"
Mac-PpBiBo $ echo ${var#*//}
www.baidu.com/index.html

# 其中 # 号是运算符，*// 表示从左边开始删除第一个 // 号及左边的所有字符
```

[参考](https://www.cnblogs.com/haona_li/p/10334057.html)

#### 4.2 数组

数组中可以存放多个值。Bash Shell 只支持一维数组（不支持多维数组），初始化时不需要定义数组大小（与 PHP 类似）。

与大部分编程语言类似，数组元素的下标由0开始。

#### 4.3 定义数组

```bash
Mac-PpBiBo $ my_array=(A B "C" D) # 注意是空格隔开而不是逗号
Mac-PpBiBo $ echo ${my_array[0]}  # 读取指定元素
A
Mac-PpBiBo $ echo ${my_array[1]}
B

Mac-PpBiBo $ a[0]=1
Mac-PpBiBo $ a[1]=2
Mac-PpBiBo $ a[2]=3
Mac-PpBiBo $ a[3]=4
Mac-PpBiBo $ echo ${a[@]}   # 读取所有元素
1 2 3 4
Mac-PpBiBo $ echo ${a[*]}   # 读取所有元素
1 2 3 4
```

#### 4.5 数组的长度

```bash
Mac-PpBiBo $ length=${my_array[*]}
Mac-PpBiBo $ echo $length
A B C D

# 第一种方法
Mac-PpBiBo $ length=${#my_array[@]}
Mac-PpBiBo $ echo $length
4

# 第二种方法
Mac-PpBiBo $ length=${#my_array[*]}
Mac-PpBiBo $ echo $length
4

# 获取数组某个元素的长度
Mac-PpBiBo $ length=${my_array[2]}
Mac-PpBiBo $ echo $length
C
Mac-PpBiBo $ length=${#my_array[2]}
Mac-PpBiBo $ echo $length
1

```

## 0x3 shell输入输出重定向

#### 1. echo

```bash
Mac-PpBiBo $ echo -e "OK one \nOK two"  # -e 开启转义
OK one 
OK two


Mac-PpBiBo $ echo '$a'    # 输出变量名使用单引号即可
$a            
Mac-PpBiBo $ echo `date`  # 输出命令执行结果使用反引号
2019年 7月14日 星期日 15时38分50秒 CST

```

#### 2. printf

```bash
printf "%-10s %-8s %-4s\n"
printf "%-10s %-8s %-4.2f\n"
printf "%-10s %-8s %-4.2f\n"
printf "%-10s %-8s %-4.2f\n"
```

*Tag：%s %c %d %f都是格式替代符%-10s指一个宽度为10个字符（-表示左对齐，没有则表示右对齐），任何字符都会被显示在10个字符宽的字符内，如果不足则自动以空格填充，超过也会将内容全部显示出来。%-4.2f指格式化为小数，其中.2指保留2位小数。*

## 0x4 shell传参

shell 代码内容

```shell
#!/bin/bash
echo $0
echo $1
echo $2
```

运行脚本并传参

```shell
./shell.sh a b
```

输出结果

```bash
./shell.sh
a
b
```

## 0x5 shell函数

#### 1. 函数定义

```shell
demoFun(){
    echo "这是我的第一个 shell 函数!"
}
echo "-----函数开始执行-----"
demoFun
echo "-----函数执行完毕-----"
```

输出

```bash
-----函数开始执行-----
这是我的第一个 shell 函数!
-----函数执行完毕-----
```

下面定义一个带有return语句的函数：

```shell
Test()
{
a="123"
return $a
}
Test
echo $?    # 函数返回值在调用该函数后通过 $? 来获得。

```

**注意**：*所有函数在使用前必须定义。这意味着必须将函数放在脚本开始部分，直至shell解释器首次发现它时，才可以使用。调用函数仅使用其函数名即可。*

#### 2. 函数参数

```shell
# 函数定义
Test()
{
echo $1
echo $2
echo $3
}
# 函数使用
Test a b c
```

##### 几个特殊字符用来处理参数：

| 参数处理 | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| $#       | 传递到脚本的参数个数                                         |
| $*       | 以一个单字符串显示所有向脚本传递的参数                       |
| $$       | 脚本运行的当前进程ID号                                       |
| $!       | 后台运行的最后一个进程的ID号                                 |
| $@       | 与$*相同，但是使用时加引号，并在引号中返回每个参数。         |
| $-       | 显示Shell使用的当前选项，与set命令功能相同。                 |
| $?       | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |

## 0x6 shell流程控制

#### 1. if条件语句

if-then-else-fi

```shell
if condition
then
    command1 
    command2
    ...
    commandN
else
    command
fi
```

if else-if else

```bash
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

```bash
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

Shell 关系运算符

```shell
关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

下表列出了常用的关系运算符，假定变量 a 为 10，变量 b 为 20：
```

| 运算符 | 说明                                                  | 举例                        |
| :----- | :---------------------------------------------------- | :-------------------------- |
| -eq    | 检测两个数是否相等，相等返回 true。                   | [ \$a -eq $b ] 返回 false。 |
| -ne    | 检测两个数是否不相等，不相等返回 true。               | [ \$a -ne $b ] 返回 true。  |
| -gt    | 检测左边的数是否大于右边的，如果是，则返回 true。     | [ \$a -gt $b ] 返回 false。 |
| -lt    | 检测左边的数是否小于右边的，如果是，则返回 true。     | [ \$a -lt $b ] 返回 true。  |
| -ge    | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [ \$a -ge $b ] 返回 false。 |
| -le    | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ \$a -le $b ] 返回 true。  |

#### 2. for循环语句

```bash
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```

```bash
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```

输出

```shell
The value is: 1
The value is: 2
The value is: 3
The value is: 4
The value is: 5
```

```shell
#!/bin/bash
for((i=1;i<=5;i++));do
    echo "这是第 $i 次调用";
done;
```

输出

```bash
这是第1次调用
这是第2次调用
这是第3次调用
这是第4次调用
这是第5次调用
```

*这里要注意一点：如果要在循环体中进行 for 中的 next 操作，记得变量要加 $，不然程序会变成死循环。*

#### 3. while 语句

```bash
while condition
do
    command
done
```

```bash
#!/bin/bash
int=1
while(( $int<=5 ))
do
    echo $int
    let "int++"
done
```

输出

```bash
1
2
3
4
5
```



[参考](https://www.runoob.com/linux/linux-shell.html)


------

***「 转载请声明博客地址及APT086&QQ愛安全实验室 」***
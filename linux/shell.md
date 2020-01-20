### 1 set & shopt



**在 Bash 中，有两个内置命令（set, shopt）用来控制 Bash 的各种可配置行为的开关（打开或关闭），这些开关称之为选项（option）。**



###### 1.1 set

set命令可以用来定制shell环境，使用选项“o”来打开或者关闭选项。

- 打开选项：set -o 选项

- 关闭选项目：set +o 选项

- 查看当前设置情况：set -o

  

常用命令：

```bash
set -eo pipefail	
```

>set -e表示一旦脚本中有命令的返回值为非0，则脚本立即退出，后续命令不再执行;
>
>set -o pipefail表示在管道连接的命令序列中，只要有任何一个命令返回非0值，则整个管道返回非0值，即使最后一个命令返回0.
>

例：
```bash
#!/bin/bash
# testset.sh
echo 'disable exit on non-zero return status and pipefail track'
set +e
set +o pipefail
a=$[1/0]|b=2
echo 'return status = '$?

echo 'disable exit on non-zero return status but enable pipefail track'
set +e
set -o pipefail
a=$[1/0]|b=2
echo 'return status = '$?

echo 'enable exit on non-zero return status and pipefail track'
set -e
set -o pipefail
a=$[1/0]|b=2
echo 'return status = '$?
```

结果：

```bash
[root@desktop2 ~]# ./testset.sh
disable exit on non-zero return status and pipefail track
./testset.sh: line 6: 1/0: division by 0 (error token is "0")
return status = 0
disable exit on non-zero return status but enable pipefail track
./testset.sh: line 12: 1/0: division by 0 (error token is "0")
return status = 1
enable exit on non-zero return status and pipefail track
./testset.sh: line 18: 1/0: division by 0 (error token is "0")
```



###### 1.2 shopt

shopt命令是set命令的一种替代，很多方面都和set命令一样，但它增加了很多选项。可有使用“-p”选项来查看shopt选项的设置。“-u”表示一unset，“-s”表示set。



常用命令

```bash
shopt -s nullglob
```

>bash允许没有匹配任何文件的文件名模式扩展成一个空串，而不是它们本身.



###### 1.3 set 和 shopt 区别

1. 在 Bash 1.* 时代，用 set 命令开启的选项只能在当前 Shell 进程中生效，没有办法通过环境变量传递给它的子进程 Shell。

2. 从 Bash 2.0 开始，新增了一个只读变量 SHELLOPTS，只要把它设置成环境变量，它就能把在当前 Shell 中打开的选项传递给子进程 Shell。

   

### 2 local & global & export



###### 2.1 local

**local一般用于局部变量声明，多在在函数内部使用。**

```bash
echo_start(){
  local STR="$1"
  echo "...... ${STR} ......starting at $(date)"
}
```



###### 2.2 global

**Shell脚本中定义的变量是global的，其作用域从被定义的地方开始，到shell结束或被显示删除的地方为止。**



###### 2.3 export

**export 将自定义变量设定为系统环境变量（**仅限于该次登陆操作，当前shell中有效）



### 3 shift

位置参数可以用`shift`命令左移。比如`shift 3`表示原来的`$4`现在变成`$1`，原来的`$5`现在变成`$2`等等，原来的`$1`、`$2`、`$3`丢弃，`$0`不移动。不带参数的`shift`命令相当于`shift 1`。



### 4 重定向  &>file	2>&1	1>&2

- 0表示标准输入(默认是键盘)
- 1表示标准输出(默认是屏幕)
- 2表示标准错误输出(默认是屏幕)
- `>` 默认为标准输出重定向，与 `1>` 相同。 文件不存在时会自动创建再写入，文件存在时会先删除文件中的内容再写入
- `>>` 文件不存在时会自动创建再写入，文件存在时不改变原文件内容再写入

例：

```bash
# 把 标准错误输出 重定向到 标准输出.
2>&1 

# 把 标准输出 和 标准错误输出 都重定向到文件file中
&>file
&>/dev/null
```



### 5 $(())，	$()，	``，	${} 区别

###### 5.1 命令替换	 ``， $( )

```bash
[root@localhost ~]# echo today is $(date "+%Y-%m-%d")
today is 2017-11-07
[root@localhost ~]# echo today is `date "+%Y-%m-%d"`
today is 2017-11-07
```

>在操作上，这两者都是达到相应的效果，但是$( )比较直观。建议使用。
>
>$( )的弊端是，并不是所有的类unix系统都支持这种方式，但反引号是肯定支持的。



###### 5.2 变量替换	${ }

一般情况下，$var与${var}是没有区别的，但是用${ }会比较精确的界定变量名称的范围。

```bash
[root@localhost ~]# A=Linux
[root@localhost ~]# echo $AB    #表示变量AB

[root@localhost ~]# echo ${A}B    #表示变量A后连接着B
LinuxB
```



###### 5.3  $(( )) 

$(( )) 可用于整数运算和进制转换

| 符号     | 功能                      |
| -------- | ------------------------- |
| + - * /  | 分别为加、减、乘、除      |
| %        | 余数运算                  |
| & \| ^ ! | 分别为“AND、OR、XOR、NOT” |

```bash
# 整数运算
[root@localhost ~]# echo $((2*3))
6
[root@localhost ~]# a=5;b=7;c=2
[root@localhost ~]# echo $((a+b*c))
19
[root@localhost ~]# echo $(($a+$b*$c))
19
```

```bash
#  进制转换
[root@localhost ~]# echo $((2#110))
6
[root@localhost ~]# echo $((16#2a))
42
[root@localhost ~]# echo $((8#11))
9
```



### 6  路径与文件名以及字符串

###### 6.1取路径，文件名，后缀

```bash
# 先赋值一个变量为一个路径，如下：
file=/dir1/dir2/dir3/my.file.txt

# 命令    解释    结果
${file#*/}    拿掉第一条 / 及其左边的字符串    dir1/dir2/dir3/my.file.txt
[root@localhost ~]# echo ${file#*/}
dir1/dir2/dir3/my.file.txt

${file##*/}    拿掉最后一条 / 及其左边的字符串    my.file.txt
[root@localhost ~]# echo ${file##*/}
my.file.txt

${file#*.}    拿掉第一个 . 及其左边的字符串    file.txt
[root@localhost ~]# echo ${file#*.}
file.txt

${file##*.}    拿掉最后一个 . 及其左边的字符串    txt
[root@localhost ~]# echo ${file##*.}
txt

${file%/*}    拿掉最后一条 / 及其右边的字符串    /dir1/dir2/dir3
[root@localhost ~]# echo ${file%/*}
/dir1/dir2/dir3

${file%%/*}    拿掉第一条 / 及其右边的字符串    (空值)
[root@localhost ~]# echo ${file%%/*}
(空值)

${file%.*}    拿掉最后一个 . 及其右边的字符串    /dir1/dir2/dir3/my.file
[root@localhost ~]# echo ${file%.*}
/dir1/dir2/dir3/my.file

${file%%.*}    拿掉第一个 . 及其右边的字符串    /dir1/dir2/dir3/my￼
[root@localhost ~]# echo ${file%%.*}
/dir1/dir2/dir3/my
```

记忆方法：

- \# 是去掉左边(在键盘上 # 在 $ 之左边)

- % 是去掉右边(在键盘上 % 在 $ 之右边)
- 单一符号是最小匹配;两个符号是最大匹配
- *是用来匹配不要的字符，也就是想要去掉的那部分
- 还有指定字符分隔号，与*配合，决定取哪部分



###### 6.2 取子串替换

```bash
# 先赋值一个变量为一个路径，如下：
file=/dir1/dir2/dir3/my.file.txt

# 用法，解释，结果
${file:0:5}            　　　提取最左边的 5 个字节    　　　　　　　　　　　　/dir1
${file:5:5}            　　　提取第 5 个字节右边的连续 5 个字节    　　　　　/dir2
${file/dir/path}            将第一个 dir 提换为 path    　　　　　　　　　 /path1/dir2/dir3/my.file.txt
${file//dir/path}    　　　　将全部 dir 提换为 path    　　　　　　　　　　　/path1/path2/path3/my.file.txt
${#file}    　　　　　　　　　 获取变量长度    　　　　　　　　　　　　　　　　　27    
```


###### 6.3 **根据状态为变量赋值**
| 命令                 | 解释                                                  | 备注                 |
| -------------------- | ----------------------------------------------------- | -------------------- |
| ${file-my.file.txt}  | 若 $file 没设定,则使用 my.file.txt 作传回值           | 空值及非空值不作处理 |
| ${file:-my.file.txt} | 若 $file 没有设定或为空值,则使用 my.file.txt 作传回值 | 非空值时不作处理     |
| ${file+my.file.txt}  | 若$file 设为空值或非空值,均使用my.file.txt作传回值    | 没设定时不作处理     |
| ${file:+my.file.txt} | 若 $file 为非空值,则使用 my.file.txt 作传回值         | 没设定及空值不作处理 |
| ${file=txt}          | 若 $file 没设定,则回传 txt ,并将 $file 赋值为 txt     | 空值及非空值不作处理 |
| ${file:=txt}         | 若 $file 没设定或空值,则回传 txt ,将 $file 赋值为txt  | 非空值时不作处理     |
| ${file?my.file.txt}  | 若 $file 没设定,则将 my.file.txt 输出至 STDERR        | 空值及非空值不作处理 |
| ${file:?my.file.txt} | 若 $file没设定或空值,则将my.file.txt输出至STDERR      | 非空值时不作处理     |



### 7 $

| 表达式   | 说明                                         |
| -------- | ---------------------------------------------------------- |
| $#   | 传递到脚本的参数个数                                         |
| $*   | 以一个单字符串显示所有向脚本传递的参数。 如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。 |
| $$   | 脚本运行的当前进程ID号                                       |
| $!   | 后台运行的最后一个进程的ID号                                 |
| $@   | 与$*相同，但是使用时加引号，并在引号中返回每个参数。 如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。 |
| $-   | 显示Shell使用的当前选项，与`set`命令功能相同。 |
| $?   | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |


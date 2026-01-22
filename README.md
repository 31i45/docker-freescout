# FreeScout 容器化部署，从测试到生产

## 什么是 FreeScout？

FreeScout是一款免费开源的工单系统，界面设计贴近大家常用的Gmail，上手很容易。它支持自主搭建部署，还能做容器化部署，不管是单机使用还是集群扩展都合适，能帮团队把客户咨询、工单都集中起来处理，多人共用一个收件箱协同办公也顺畅，完全能替代那些收费的同类工具。

在客户支持领域，Zendesk和Help Scout是公认的标杆，但它们不仅使用成本高，还存在潜在隐私风险，不少中小企业都觉得负担不起或不放心。而FreeScout作为轻量级开源方案，基于PHP（Laravel框架）开发，刚好补上了这个缺口——免费可用、自己掌控数据，隐私安全有保障，容器化部署还能快速适配企业现有云环境，简化集群管理、提升部署一致性。

它从一开始就独立开发，没用到任何受版权保护的素材，合规性和独立性都没问题。不管是小初创公司，还是规模较大的企业，它的功能都能适配，日常客户支持需求基本都能覆盖。

作为社区驱动的项目，FreeScout鼓励用户参与开发、提功能建议，迭代更新也比较贴合实际使用需求。综合来看，不管是控制成本、保护数据隐私，还是拓展功能、灵活部署，它都是很能打的开源客户支持工具。

## 项目特点：

* **无数量限制**：用户、工单、绑定邮箱都没有上限，业务增长不用受工具限制。

* **移动适配性强**：完全支持各类移动设备，外出也能随时处理客户咨询，不耽误工作。

* **多语言支持**：内置多种语言，面向全球客户提供支持也没压力。

* **安全有保障**：自带完善的安全机制，能有效保护用户和客户的数据安全。

* **部署简单高效**：提供Web安装程序和自动更新工具，搭配容器化部署，跨环境迁移、集群运维更省心，还能降低兼容成本。

* **模块化拓展**：支持官方和社区开发的拓展模块，能根据自己的业务需求增减功能。

* **多元集成能力**：提供API接口，还能对接Zapier等工具，轻松和企业现有系统联动协作。

* **离线部署+虚拟邮箱适配**：支持离线使用，可通过虚假邮箱创建账号，工单以虚假邮箱作为客户信息记录全程，无需依赖真实互联网邮箱或内网邮件服务器。

---

下文中，我将按 “测试→准生产→生产级” 的逻辑，详细说明 3 种容器化部署方式，可无缝适配从测试到轻度生产环境的全部需求。如果你有更复杂的需求，请自行探索和扩展，比如内网穿透和域名，可以参考我的这篇公众号文章[Cloudflare隧道：将家庭网络安全地暴露到互联网上 https://mp.weixin.qq.com/s/cjXsfvKsb3uHB2lBRT-0iQ](https://mp.weixin.qq.com/s/cjXsfvKsb3uHB2lBRT-0iQ)，或者[本地部署开源客服系统 FreeScout 并实现外部访问 https://help.luyouxia.com/freescout-windows.html](https://help.luyouxia.com/freescout-windows.html )。

## 参考链接：

[https://github.com/freescout-help-desk/freescout/wiki/Installation-Guide](https://github.com/freescout-help-desk/freescout/wiki/Installation-Guide)

[https://github.com/tiredofit/docker-freescout/blob/main/examples/compose-no-proxy.yml](https://github.com/tiredofit/docker-freescout/blob/main/examples/compose-no-proxy.yml)

[https://github.com/nfrastack/container-mariadb](https://github.com/nfrastack/container-mariadb)

[https://github.com/tiredofit/docker-freescout/blob/main/examples/compose.yml](https://github.com/tiredofit/docker-freescout/blob/main/examples/compose.yml)

## 局域网离线测试部署（HTTP 访问）

```bash
[student@rocky ~]$ cd freescout/
[student@rocky freescout]$ pwd
/home/student/freescout
[student@rocky freescout]$ ls
compose-no-proxy.yml  compose.yml 
[student@rocky freescout]$ 
[student@rocky freescout]$ cat compose-no-proxy.yml
services:
  freescout-app:
    image: tiredofit/freescout
    container_name: freescout-app
    ports:
    - 8080:80
    links:
    - freescout-db
    volumes:
    ### 需自定义修改并访问源代码时，取消该行注释（含模块场景）。
    #- ./data:/www/html
    ### 若仅使用原生FreeScout，且需保留缓存、会话等持久化文件，用下面这行配置。
    - ./data:/data
    ### 若需保留原始源代码，同时添加额外模块，取消下面行注释。
    #- ./modules:/www/html/Modules
    - ./logs/:/www/logs
    environment:
    - CONTAINER_NAME=freescout-app

    - DB_HOST=freescout-db
    - DB_NAME=freescout
    - DB_USER=freescout
    - DB_PASS=freescout

    - SITE_URL=http://192.168.0.115:8080
    - ADMIN_EMAIL=admin@admin.com
    - ADMIN_PASS=freescout
    - ENABLE_SSL_PROXY=FALSE
    - DISPLAY_ERRORS=FALSE
    - TIMEZONE=Asia/Shanghai
    restart: always

  freescout-db:
    #image: tiredofit/mariadb    # 这个mariadb版本太老
    image: docker.io/nfrastack/mariadb
    container_name: freescout-db
    volumes:
      - ./db:/data
    environment:
      - ROOT_PASS=password
      - DB_NAME=freescout
      - DB_USER=freescout
      - DB_PASS=freescout

      - TIMEZONE=Asia/Shanghai    # 指定时区对任何应用都很重要

      - CONTAINER_NAME=freescout-db
    restart: always

  freescout-db-backup:
    container_name: freescout-db-backup
    image: tiredofit/db-backup
    links:
     - freescout-db
    volumes:
      - ./dbbackup:/backup
    environment:
      - CONTAINER_NAME=freescout-db-backup
      - DB_HOST=freescout-db
      - DB_TYPE=mariadb
      - DB_NAME=freescout
      - DB_USER=freescout
      - DB_PASS=freescout
      - DB01_BACKUP_INTERVAL=1440
      - DB01_BACKUP_BEGIN=0000
      - DB_CLEANUP_TIME=8640
      - COMPRESSION=BZ
      - MD5=TRUE

      - TIMEZONE=Asia/Shanghai
    restart: always
[student@rocky freescout]$ docker compose -f compose-no-proxy.yml up -d
[student@rocky freescout]$ docker compose -f compose-no-proxy.yml logs -
[student@rocky freescout]$ curl -vvv http://192.168.0.115:8080
```

## 局域网离线测试部署（HTTPS 访问）

```bash
[student@rocky freescout]$
[student@rocky freescout]$ cat compose.yml
services:

  freescout-app:
    image: docker.io/tiredofit/freescout
    container_name: freescout-app
    volumes:
      - ./data:/data
      - ./logs/:/www/logs
    environment:
      - TIMEZONE=Asia/Shanghai
      - CONTAINER_NAME=freescout-app

      - DB_HOST=freescout-db
      - DB_NAME=freescout
      - DB_USER=freescout
      - DB_PASS=freescout

      - SITE_URL=https://192.168.0.115

      - ADMIN_EMAIL=admin@admin.com
      - ADMIN_PASS=freescout

      - DISPLAY_ERRORS=FALSE

      - ENABLE_SSL_PROXY=TRUE
    networks:
      - proxy
      - services
    restart: always

  freescout-db:
    image: docker.io/nfrastack/mariadb
    container_name: freescout-db
    volumes:
      - ./db:/data
    environment:
      - TIMEZONE=Asia/Shanghai
      - CONTAINER_NAME=freescout-db

      - ROOT_PASS=password
      - DB_NAME=freescout
      - DB_USER=freescout
      - DB_PASS=freescout
    networks:
      - services
    restart: always

  freescout-db-backup:
    container_name: freescout-db-backup
    image: docker.io/tiredofit/db-backup
    volumes:
      - ./dbbackup:/backup
    environment:
      - TIMEZONE=Asia/Shanghai
      - CONTAINER_NAME=freescout-db-backup

      - DB01_HOST=freescout-db
      - DB01_TYPE=mariadb
      - DB01_NAME=freescout
      - DB01_USER=freescout
      - DB01_PASS=freescout
      - DB01_BACKUP_INTERVAL=1440
      - DB01_BACKUP_BEGIN=0000
      - DB01_CLEANUP_TIME=8640
    networks:
      - services
    restart: always

networks:
  proxy:
    #external: true    # 注释掉这两行，部署容器时才能自动创建网络
  services:
    #external: true
```

我们需要部署 Caddy 容器，以实现 HTTPS 反向代理功能，并自动生成自签名证书。

```bash
[student@rocky freescout]$ # 创建Caddy目录+设置权限（仅需执行一次）
[student@rocky freescout]$ mkdir -p ./caddy/{conf,data,logs} && chown -R 1000:1000 ./caddy/{data,logs} && chmod 755 ./caddy/{conf,data,logs}
[student@rocky freescout]$ tree caddy/
caddy/
├── conf
├── data
└── logs
3 directories, 0 files
[student@rocky freescout]$ cat caddy/caddy.yml
services:
  caddy:
    image: caddy:2.10.0-alpine
    container_name: freescout-caddy
    volumes:
      - ./conf/Caddyfile:/etc/caddy/Caddyfile  # 挂载配置文件
      - ./data:/data                           # 证书/数据持久化
      - ./logs:/var/log/caddy                  # 日志持久化
    ports:
      - "80:80"   # HTTP端口
      - "443:443" # HTTPS端口
    networks:
      - freescout_proxy    # 加入Freescout的proxy网络（关键！）
    restart: always  # 开机自启

# 加入Freescout创建的proxy网络（必须和Freescout的网络名一致）
networks:
  freescout_proxy:    # 网络名称可以用 docker network ls 命令查看
    external: true  # 表示该网络已由Freescout创建，直接复用
[student@rocky freescout]$ cat caddy/conf/Caddyfile
# 全局配置
{
    default_sni 192.168.0.115  # 替换为你的服务器内网IP
    servers :443 {
        protocols h1 h2 h3
    }
}

# HTTPS反向代理（指向Freescout）
192.168.0.115:443 {
    tls internal  # 自签名证书 也可以使用 Let’s Encrypt 申请的免费证书

    # 安全头（可选但推荐）
    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains"
        X-Content-Type-Options "nosniff"
        X-Frame-Options "SAMEORIGIN"
        -Server
    }

    # 反向代理到Freescout的容器（核心）
    reverse_proxy freescout-app:80 {
        transport http {
            keepalive 3600s
            dial_timeout 10s
        }
        header_up X-Forwarded-Proto {scheme}
        header_up X-Forwarded-For {remote}
        header_up Host {host}
    }

    # 日志配置
    log {
        output file /var/log/caddy/freescout-access.log {
            roll_size 100MB
            roll_keep 5
        }
        format json
    }
}

# HTTP强制跳HTTPS
http://192.168.0.115 {
    redir https://{host}{uri} 301
}
[student@rocky freescout]$ docker compose -f compose.yml up -d
[student@rocky freescout]$ docker compose -f caddy/caddy.yml up -d
[student@rocky freescout]$ docker compose -f caddy/caddy.yml ps
[student@rocky freescout]$ docker compose -p freescout logs -f
[student@rocky freescout]$ docker compose -p caddy logs -f
[student@rocky freescout]$ curl -vvv -k https://192.168.0.115
*   Trying 192.168.0.115:443...
* Connected to 192.168.0.115 (192.168.0.115) port 443 (#0)
# 省略一些输出
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <meta http-equiv="refresh" content="0;url=https://192.168.0.115/login" />

        <title>Redirecting to https://192.168.0.115/login</title>
    </head>
    <body>
        Redirecting to <a href="https://192.168.0.115/login">https://192.168.0.115/login</a>.
    </body>
* TLSv1.2 (IN), TLS header, Unknown (23):
* Connection #0 to host 192.168.0.115 left intact
</html>
[student@rocky freescout]$ # http方式无法访问
[student@rocky freescout]$ curl -vvv http://192.168.0.115
*   Trying 192.168.0.115:80...
* Connected to 192.168.0.115 (192.168.0.115) port 80 (#0)
> GET / HTTP/1.1
> Host: 192.168.0.115
> User-Agent: curl/7.76.1
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 301 Moved Permanently
< Location: https://192.168.0.115/
< Server: Caddy
< Date: Wed, 21 Jan 2026 02:13:07 GMT
< Content-Length: 0
<
* Connection #0 to host 192.168.0.115 left intact
[student@rocky freescout]$
```

## 复杂生产场景：云主机的EIP（HTTPS 访问）

EIP 实现公网通信的核心依赖 NAT 机制的地址转换能力，因此，下面这种 freescout 的部署场景也适合其他 配置了 NAT 地址转换的场合。虚拟机安全组记得放通相应的端口。

其中，Caddyfile 的配置参考了我的另一篇公众号文章 [更新：NextCloud 自动部署脚本 https://mp.weixin.qq.com/s/s5KehVjxZyzU1lO4avtG2w](https://mp.weixin.qq.com/s/s5KehVjxZyzU1lO4avtG2w)。

```bash
[root@devops freescout]# cat freescout-with-proxy.yml
services:

  freescout-app:
    image: docker.io/tiredofit/freescout
    container_name: freescout-app
    volumes:
      - ./data:/data
      - ./logs/:/www/logs
    environment:
      - TIMEZONE=Asia/Shanghai
      - CONTAINER_NAME=freescout-app

      - DB_HOST=freescout-db
      - DB_NAME=freescout
      - DB_USER=freescout
      - DB_PASS=freescout

      - SITE_URL=https://225.105.121.8:28443  # 替换为你的实际EIP
      #- SITE_URL=http://freescout-app:80
      - TRUSTED_PROXIES=0.0.0.0/0

      - ADMIN_EMAIL=admin@admin.com
      - ADMIN_PASS=freescout

      - DISPLAY_ERRORS=FALSE

      - ENABLE_SSL_PROXY=TRUE
      #- ENABLE_SSL_PROXY=FALSE
    networks:
      - proxy
      - services
    restart: always

  freescout-db:
    image: docker.io/nfrastack/mariadb
    container_name: freescout-db
    volumes:
      - ./db:/data
    environment:
      - TIMEZONE=Asia/Shanghai
      - CONTAINER_NAME=freescout-db

      - ROOT_PASS=password
      - DB_NAME=freescout
      - DB_USER=freescout
      - DB_PASS=freescout
    networks:
      - services
    restart: always

  freescout-db-backup:
    container_name: freescout-db-backup
    image: docker.io/tiredofit/db-backup
    volumes:
      - ./dbbackup:/backup
    environment:
      - TIMEZONE=Asia/Shanghai
      - CONTAINER_NAME=freescout-db-backup

      - DB01_HOST=freescout-db
      - DB01_TYPE=mariadb
      - DB01_NAME=freescout
      - DB01_USER=freescout
      - DB01_PASS=freescout
      - DB01_BACKUP_INTERVAL=1440
      - DB01_BACKUP_BEGIN=0000
      - DB01_CLEANUP_TIME=8640
    networks:
      - services
    restart: always

  caddy:
    image: caddy:2.10.0-alpine
    container_name: freescout-caddy
    environment:
      - TIMEZONE=Asia/Shanghai
    volumes:
      - ./caddy/conf/Caddyfile:/etc/caddy/Caddyfile  # 挂载配置文件
      - ./caddy/data:/data                           # 证书/数据持久化
      - ./caddy/logs:/var/log/caddy                  # 日志持久化
    ports:
      - "28080:80"   # HTTP端口
      - "28443:443" # HTTPS端口
    networks:
      - proxy    # 加入Freescout的proxy网络（关键！）
    restart: always  # 开机自启

networks:
  proxy:
  services:

[root@devops freescout]# docker exec -it freescout-caddy cat /etc/caddy/Caddyfile
{
    # 保留Nextcloud成功的全局配置（解决TLS握手问题的核心）
    default_sni 225.105.121.8  # 替换为你的实际EIP
    servers :443 {
        protocols h1 h2 h3
    }
}

# 核心：监听225.105.121.8:443（容器内），对应宿主机28443端口
225.105.121.8:443 {
    # 保留稳定的TLS配置（复用Nextcloud的成功配置）
    tls internal

    # 保留通用安全头（无需改动）
    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains"
        X-Content-Type-Options "nosniff"
        X-Frame-Options "SAMEORIGIN"
        -Server
    }

    # 保留压缩（无需改动）
    encode gzip

    # ========== 仅新增：Freescout反向代理（核心改动） ==========
    reverse_proxy freescout-app:80 {
        # 传递外部访问上下文（解决225.105.121.8:28443→freescout-app:80的地址映射）
        header_up Host {host}:28443
        header_up X-Forwarded-Proto https
        header_up X-Forwarded-For {remote_host}
        header_up X-Forwarded-Port 28443

        # 保留长连接支持（复用Nextcloud的可靠配置）
        transport http {
            keepalive 300s
            read_timeout 3600s
            write_timeout 3600s
            dial_timeout 10s
        }
    }

    # 日志配置
    log {
        output file /var/log/caddy/freescout-access.log {
            roll_size 100MB
            roll_keep 5
        }
        format json
    }
}

# HTTP重定向：适配28080端口，重定向到HTTPS的28443
http://225.105.121.8 {  # 替换为你的实际EIP
    redir https://{host}:28443{uri} 301
}
[root@devops freescout]#
[student@rocky freescout]$ docker compose -f freescout-with-proxy.yml up -d
[student@rocky freescout]$ docker compose -f freescout-with-proxy.yml ps
[student@rocky freescout]$ docker compose -f freescout-with-proxy.yml logs -f
[student@rocky freescout]$ curl -vvv -k https://225.105.121.8:28443
```

## The End

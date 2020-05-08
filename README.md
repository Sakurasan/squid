[目录]

> squid:/etc/squid3/squid.conf

> squid:/var/log/squid3

## 一条命令梭哈
```
docker run --name squid -d --restart=always \
  -p 3128:3128 \
  -v $CODEPATH/squid/cache:/var/spool/squid3 \
  -v $CODEPATH/squid/log:/var/log/squid3 \
  sameersbn/squid
```

##  生成认证文件
```
 htpasswd squid_passwd your-username
```
> 在这里输入两次密码

> 将认证文件拷贝至容器
```
 docker cp squid_passwd squid:/etc/squid3/
```

## [简易配置](squid-simple.conf)
```
acl localnet src 10.0.0.0/8    # RFC1918 possible internal network
acl localnet src 172.16.0.0/12    # RFC1918 possible internal network
acl localnet src 192.168.0.0/16    # RFC1918 possible internal network
acl localnet src fc00::/7       # RFC 4193 local private network range
acl localnet src fe80::/10      # RFC 4291 link-local (directly plugged) machines
acl localnet src 0.0.0.0/0.0.0.0
acl localnet src 0.0.0.0/8

acl SSL_ports port 443
acl Safe_ports port 80        # http
acl Safe_ports port 21        # ftp
acl Safe_ports port 443        # https
acl Safe_ports port 70        # gopher
acl Safe_ports port 210        # wais
acl Safe_ports port 1025-65535    # unregistered ports
acl Safe_ports port 280        # http-mgmt
acl Safe_ports port 488        # gss-http
acl Safe_ports port 591        # filemaker
acl Safe_ports port 777        # multiling http
acl CONNECT method CONNECT

# username&password auth config
auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid3/squid_passwd
acl ncsa_users proxy_auth REQUIRED
http_access allow ncsa_users


http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localhost manager
http_access deny manager
http_access deny to_localhost
http_access allow localnet
http_access allow localhost
http_access deny all
http_port 3128

cache_dir ufs /var/spool/squid3 100 16 256
coredump_dir /var/spool/squid3

refresh_pattern ^ftp:        1440    20%    10080
refresh_pattern ^gopher:    1440    0%    1440
refresh_pattern -i (/cgi-bin/|\?) 0    0%    0
refresh_pattern (Release|Packages(.gz)*)$      0       20%     2880
refresh_pattern .        0    20%    4320
```
### 将配置文件导入Squid容器
```
docker cp squid-simple.conf squid:/etc/squid3/squid.conf
```

## 配置用户名密码认证

- 生成认证文件
    ```
    $ htpasswd squid_passwd your-username
    ```
    > 在这里输入两次密码

- 将认证文件拷贝至容器
    ```
    $ docker cp squid_passwd squid:/etc/squid3/
    ```

## 生成配置文件说明
```
docker cp squid-simple.conf squid:/etc/squid3/squid.conf && \
awk '/^[^#]/' squid.conf > squid2.conf

vim squid-simple.conf
## 在这里添加几行
## acl localnet src 0.0.0.0/0.0.0.0
## acl localnet src 0.0.0.0/8

#### username&password auth config
## auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid3/squid_passwd
## acl ncsa_users proxy_auth REQUIRED
## http_access allow ncsa_users
```

---

参考:
- https://segmentfault.com/a/1190000014388375#item-5
- https://www.cnblogs.com/wangxiaoqiangs/p/5796624.html


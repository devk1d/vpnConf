### Debian 8 系统


#### 1. 安装 BBR

```
wget --no-check-certificate https://github.com/iyuco/scripts/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
```
一路回车，看到 **Y/n** 选择 **n**


#### 2. 安装 shadowsocksR 一键脚本

```
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh && chmod +x shadowsocksR.sh
./shadowsocksR.sh
```

一路回车，配置文件位置:

```
/etc/shadowsocks.json
```

多用户配置示例：

```
{
    "server":"0.0.0.0",
    "server_ipv6":"[::]",
    "local_address":"127.0.0.1",
    "local_port":1080,
    "port_password": {
      "10000": "test",
      "10001": "test",
    },
    "timeout":120,
    "method":"aes-256-cfb",
    "protocol":"origin",
    "protocol_param":"",
    "obfs":"plain",
    "obfs_param":"",
    "redirect":"",
    "dns_ipv6":false,
    "fast_open":false,
    "workers":1
}
```


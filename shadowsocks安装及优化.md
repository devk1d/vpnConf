

## 安装

apt-get install python-pip
pip install shadowsocks

## 创建配置

vi /etc/shadowsocks.json

配置内容：

	{
	  "server":"服务器IP",

	  "local_port": 1080,

	  "port_password": {

	    "10000": "password",

	    "10001": "password",
	  },

	  "timeout": 60,

	  "fast_open": true,

	  "method": "aes-256-cfb"
	}

## 启动服务

先安装 screen
		
	apt-get install screen

然后 
		
	screen -R
	ssserver -c /etc/shadowsocks.json

Ctrl + c 退出 screen
		

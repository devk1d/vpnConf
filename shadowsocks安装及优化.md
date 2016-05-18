

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

## 优化

第一步，增加系统文件描述符的最大限数

编辑文件 limits.conf

	vi /etc/security/limits.conf

增加以下两行

	soft nofile 51200
	hard nofile 51200

启动shadowsocks服务器之前，设置以下参数

	ulimit -n 51200

第二步，调整内核参数
修改配置文件 /etc/sysctl.conf

	fs.file-max = 51200

	net.core.rmem_max = 67108864
	net.core.wmem_max = 67108864
	net.core.netdev_max_backlog = 250000
	net.core.somaxconn = 4096

	net.ipv4.tcp_syncookies = 1
	net.ipv4.tcp_tw_reuse = 1
	net.ipv4.tcp_tw_recycle = 0
	net.ipv4.tcp_fin_timeout = 30
	net.ipv4.tcp_keepalive_time = 1200
	net.ipv4.ip_local_port_range = 10000 65000
	net.ipv4.tcp_max_syn_backlog = 8192
	net.ipv4.tcp_max_tw_buckets = 5000
	net.ipv4.tcp_fastopen = 3
	net.ipv4.tcp_rmem = 4096 87380 67108864
	net.ipv4.tcp_wmem = 4096 65536 67108864
	net.ipv4.tcp_mtu_probing = 1
	net.ipv4.tcp_congestion_control = hybla
	
修改后执行 sysctl -p 使配置生效

## 安装net-speeder

分别执行以下命令：

	wget --no-check-certificate https://gist.github.com/LazyZhu/dc3f2f84c336a08fd6a5/raw/d8aa4bcf955409e28a262ccf52921a65fe49da99/net_speeder_lazyinstall.sh

	sh net_speeder_lazyinstall.sh

加入开机启动

	echo 'nohup /usr/local/net_speeder/net_speeder venet0 "ip" >/dev/null 2>&1 &' >> /etc/rc.local

## 重启VPS

	reboot

## 启动服务

先安装 screen
		
	apt-get install screen

然后 
		
	screen -R
	ssserver -c /etc/shadowsocks.json

Ctrl + c 退出 screen
		

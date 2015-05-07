用的是linode，基于Unbuntu 12.04LTS 64位，32位的没试过，装好系统后设置

PPTP:

安装pptpd

	apt-get update
	apt-get install pptpd
	
编辑pptpd-options

	vi /etc/ppp/pptpd-options
	
底部添加
	
	nopcomp
	noaccomp

	ms-dns 8.8.8.8
	ms-dns 8.8.4.4

编辑pptd.conf

	vi /etc/pptpd.conf
	
底部添加

	localip 192.168.17.1
	remoteip 192.168.17.2-200

编辑options
	
	vi /etc/ppp/options
	
底部添加

	ms-dns 8.8.8.8
	ms-dns 8.8.4.4
	
编辑

	vi /etc/ppp/chap-secrets

添加用户

	test * test *
	
编辑

	vi /etc/sysctl.conf
	
底部添加

	net.ipv4.ip_forward=1
	net.ipv4.conf.all.accept_redirects = 0
	net.ipv4.conf.all.send_redirects = 0
	net.ipv4.tcp_syncookies = 1
	
执行
	
	sysctl -p
	
编辑
	
	vi /etc/rc.local

在exit 0前添加，$ip 替换为你的vps ip地址

	iptables -t nat -A POSTROUTING -j SNAT --to $ip
	iptables -t nat -A POSTROUTING -s 192.168.17.0/24 -o eth0 -j MASQUERADE
	sysctl -p
	
	
命令行执行，$ip 替换为你的vps ip地址

	iptables -t nat -A POSTROUTING -j SNAT --to $ip
	iptables -t nat -A POSTROUTING -s 192.168.17.0/24 -o eth0 -j MASQUERADE
	iptables-save

执行
	
	service pptpd restart




L2TP:

	
安装相应软件
	
	apt-get install openswan xl2tpd ppp lsof

安装openswan时可能会弹出问题让你确认，一直回车就是.

编辑ipsec

	vi /etc/ipsec.conf
	
注释掉oe=off
	
	#oe=off

protostack设置为netkey

	protostack=netkey
	
底部添加，$ip 替换为你的vps ip地址

	conn %default
		forceencaps=yes

	conn L2TP-PSK-NAT
		rightsubnet=vhost:%no,%priv
		also=L2TP-PSK-noNAT

	conn L2TP-PSK-noNAT
		authby=secret
		pfs=no
		auto=add
		keyingtries=3
		rekey=no
		ikelifetime=8h
		keylife=1h
		type=transport
		left=$ip
		leftprotoport=17/1701
		right=%any
		rightprotoport=17/%any
	
编辑ipsec.secrets

		vi /etc/ipsec.secrets
		
写入, $ip 替换为你的vps ip地址，test替换为你想设的密钥
		
	$ip %any: PSK "test"
	
编辑xl2tpd.conf

	vi /etc/xl2tpd/xl2tpd.conf
	
写入

	[global]
		; listen-addr = 192.168.1.98

	[lns default]
		ip range = 192.168.15.2-192.168.15.200
		local ip = 192.168.15.1
		require chap = yes
		refuse pap = yes
		require authentication = yes
		name = LinuxVPNserver
		ppp debug = yes
		pppoptfile = /etc/ppp/options.xl2tpd
		length bit = yes
		
执行
	
	cp /usr/share/doc/xl2tpd/examples/ppp-options.xl2tpd \
		/etc/ppp/options.xl2tpd
		
编辑

	vi /etc/ppp/options.xl2tpd
	
把ms-dns替换为

	ms-dns 8.8.8.8
	ms-dns 8.8.4.4
	
	
编辑rc.local

	vi /etc/rc.local
	

在exit 0前添加

	for vpn in /proc/sys/net/ipv4/conf/*; do echo 0 > $vpn/accept_redirects; echo 0 > $vpn/send_redirects; done
		iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -o eth0 -j MASQUERADE
		iptables --table nat --append POSTROUTING --jump MASQUERADE
		

执行

	for vpn in /proc/sys/net/ipv4/conf/*; do echo 0 > $vpn/accept_redirects; echo 0 > $vpn/send_redirects; done
	iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -o eth0 -j MASQUERADE
	iptables --table nat --append POSTROUTING --jump MASQUERADE

	iptables-save
	
	
执行
	ipsec verify
	service xl2tpd restart
	service ipsec restart
	
	

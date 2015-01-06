L2TP:

	
1. 先更新apt
	
		apt-get update
	
2. 安装相应软件
	
		apt-get install openswan xl2tpd ppp lsof

	安装openswan时可能会弹出问题让你确认，一直回车就是.
	
3. 设置防火墙，允许vpn访问

		iptables --table nat --append POSTROUTING --jump MASQUERADE
	
4. 允许ip转发

		echo "net.ipv4.ip_forward = 1" |  tee -a /etc/sysctl.conf
		echo "net.ipv4.conf.all.accept_redirects = 0" |  tee -a /etc/sysctl.conf
		echo "net.ipv4.conf.all.send_redirects = 0" |  tee -a /etc/sysctl.conf
		for vpn in /proc/sys/net/ipv4/conf/*; do echo 0 > $vpn/accept_redirects; echo 0 > $vpn/send_redirects; done
		sysctl -p
		
	这一大串复制到命令行直接运行
	
	
5. 重启后3、4步会失效，所有要在开机启动时执行命令
	
	编辑rc.local
		
		vi /etc/rc.local
	以下内容添加到文件中，如果文件中有 exit 0 的话，加到 exit 0 前面
	
		for vpn in /proc/sys/net/ipv4/conf/*; do echo 0 > $vpn/accept_redirects; echo 0 > $vpn/send_redirects; done
		iptables --table nat --append POSTROUTING --jump MASQUERADE
		
6. 编辑ipsec.conf
	
		vi /etc/ipsec.conf

	删除原有内容，写入（%SERVERIP% 改成你的vps ip地址！）
		
		config setup
    		dumpdir=/var/run/pluto/
    		#in what directory should things started by setup (notably the Pluto daemon) be allowed to dump core?
    		nat_traversal=yes
    		#whether to accept/offer to support NAT (NAPT, also known as "IP Masqurade") workaround for IPsec
    		virtual_private=%v4:10.0.0.0/8,%v4:192.168.0.0/16,%v4:172.16.0.0/12,%v6:fd00::/8,%v6:fe80::/10
    		#contains the networks that are allowed as subnet= for the remote client. In other words, the address ranges that may live behind a NAT router through which a client connects.
		    protostack=netkey
		    #decide which protocol stack is going to be used.

		conn L2TP-PSK-NAT
		    rightsubnet=vhost:%priv
		    also=L2TP-PSK-noNAT

		conn L2TP-PSK-noNAT
		    authby=secret
		    #shared secret. Use rsasig for certificates.
		    pfs=no
		    #Disable pfs
		    auto=add
		    #start at boot
		    keyingtries=3
		    #Only negotiate a conn. 3 times.
		    ikelifetime=8h
		    keylife=1h
		    type=transport
		    #because we use l2tp as tunnel protocol
		    left=%SERVERIP%
		    #fill in server IP above
		    leftprotoport=17/1701
		    right=%any
		    rightprotoport=17/%any
		    
		    
7. 设置ipsec安全密钥
	
		 vi /etc/ipsec.secrets
		 
	写入：（%SERVERIP% 依然换成你的IP，test是你的密钥，改成你想设置的任意字符）
	 
		%SERVERIP%  %any:   PSK "test"

8. ipsec验证

		 ipsec verify
		
	我的输出
		
		Checking your system to see if IPsec got installed and started correctly:
		Version check and ipsec on-path                                 [OK]
		Linux Openswan U2.6.37/K3.2.0-29-generic-pae (netkey)
		Checking for IPsec support in kernel                            [OK]
		 SAref kernel support                                           [N/A]
		 NETKEY:  Testing XFRM related proc values                      [OK]
		    [OK]
		    [OK]
		Checking that pluto is running                                  [OK]
		 Pluto listening for IKE on udp 500                             [OK]
		 Pluto listening for NAT-T on udp 4500                          [OK]
		Two or more interfaces found, checking IP forwarding            [OK]
		Checking NAT and MASQUERADEing                                  [OK]
		Checking for 'ip' command                                       [OK]
		Checking /bin/sh is not /bin/dash                               [WARNING]
		Checking for 'iptables' command                                 [OK]
		Opportunistic Encryption Support                                [DISABLED]
				
	如果是差不多这样的，就没问题
	
9. 配置xl2tp


		vi /etc/xl2tpd/xl2tpd.conf
	
	删掉之前的，写入
	
		[global]
			ipsec saref = yes

		[lns default]
			ip range = 172.16.1.30-172.16.1.100
			local ip = 172.16.1.1
			refuse pap = yes
			require authentication = yes
			ppp debug = yes
			pppoptfile = /etc/ppp/options.xl2tpd
			length bit = yes
	
	
10. 写入PPP

		vi /etc/ppp/options.xl2tpd
		
	写入
		
		require-mschap-v2
		ms-dns 8.8.8.8
		ms-dns 8.8.4.4
		auth
		mtu 1200
		mru 1000
		crtscts
		hide-password
		modem
		name l2tpd
		proxyarp
		lcp-echo-interval 30
		lcp-echo-failure 4

11. 增加用户

		vi /etc/ppp/chap-secrets

	仿造下面的形式

		username1          *   password1            *
		username2          l2tpd   password2             *

	这样username1支持任何协议，pptp和l2tpd，username2只支持l2tpd

12. 最后
		
		/etc/init.d/ipsec restart;  /etc/init.d/xl2tpd restart


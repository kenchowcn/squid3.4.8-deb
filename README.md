squid3.4.8-deb
=====

I built those packages fellow [squid-3-4-x-with-ssl-for-debian-wheezy](http://codepoets.co.uk/2014/squid-3-4-x-with-ssl-for-debian-wheezy/) and [how-to-filter-https-traffic-with-squid-3-on-ubuntu-server-13-10](http://ubuntuserverguide.com/2013/12/how-to-filter-https-traffic-with-squid-3-on-ubuntu-server-13-10.html), if you have any questions or why, please find reason from those two articles by yourself, I only introduce how to install and make them works on **Debian 8 64bit**.

**if you understand what is each command for, do it as you think, or don't think too much just copy & paste & execute.**

### Clone

	git clone https://github.com/kenchowcn/squid3.4.8-deb.git

###Install DEB

	sudo apt-get --purge remove squid3*
	sudo apt-get -y install squid-langpack
	sudo apt-get -y build-dep squid3
	sudo dpkg -i squid3*.deb

###Make CA

	sudo mkdir -p /etc/squid3/ssl && cd /etc/squid3/ssl
	sudo openssl genrsa -out squid.key 2048
	sudo openssl req -new -key squid.key -out squid.csr
	sudo openssl x509 -req -days 3650 -in squid.csr -signkey squid.key -out squid.crt
	sudo cat squid.key squid.crt > squid.pem

###Edit squid3.conf	

Find "http_port 3128" in /etc/squid3/squid3.conf, and add it.

	# for clients with a configured proxy.
	http_port 3128
	# for clients who are sent here via iptables ... REDIRECT.
	http_port 3129 intercept
	# for https clients who are sent here via iptables ... REDIRECT
	https_port 3130 intercept ssl-bump  generate-host-certificates=on dynamic_cert_mem_cache_size=4MB cert=/etc/squid3/ssl/squid.pem

	always_direct allow all
	ssl_bump none localhost
	ssl_bump server-first all
	sslproxy_cert_error allow all
	sslproxy_flags DONT_VERIFY_PEER

###Iptables setting (for router, also for PC, watch out "ethX")

	iptables -F
	iptables -X
	iptables -t nat -F
	iptables -t nat -X
	iptables -t mangle -F
	iptables -t mangle -X

	iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 3129
	iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 80 -j DNAT --to-destination 127.0.0.1:3129

	iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 3130
	iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 443 -j DNAT --to-destination 127.0.0.1:3130
	
###Run Squid3	
	
	/usr/lib/squid3/ssl_crtd -c -s /var/lib/ssl_db/
	chown -R proxy /var/lib/ssl_db
	service squid3 restart
	
###Verify
	
1. Install squid.crt to your system or browser(firefox), please see the step "Make CA" to find squid.crt.

2. Enable proxy in your browser, proxy IP is your squid3's host IP, and port is 3128.
	


---
layout: post
title: shadowsocks一键安装脚本
category: project
description: 适用于centos、ubuntu、debian系统
---

# github repository url：

[https://github.com/jekyllcao/shadowsocks-install-script-for-centos-ubuntu-debian][1]


# 可用脚本

	
	!/usr/bin/env bash
	PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
	export PATH
	=================================================================#
	  System Required:  CentOS 6+, Debian 7+, Ubuntu 12+            #
	  Description: One click Install Shadowsocks-Python server      #
	  Author: Teddysun <i@teddysun.com>                             #
	  Thanks: @clowwindy <https://twitter.com/clowwindy>            #
	  Intro:  https://teddysun.com/342.html                         #
	=================================================================#
	
	clear
	echo
	echo "#############################################################"
	echo "# One click Install Shadowsocks-Python server               #"
	
	echo "# Modifier: Eric  @refrer:teddysun.com                      #"
	echo "# Github: https://github.com/shadowsocks/shadowsocks        #"
	echo "#############################################################"
	echo
	
	Current folder
	cur_dir=pwd
	
	Get public IP address
	get_ip(){
	local IP=$( ip addr | egrep -o '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | egrep -v "^192\.168|^172\.1[6-9]\.|^172\.2[0-9]\.|^172\.3[0-2]\.|^10\.|^127\.|^255\.|^0\." | head -n 1 )
	[ -z ${IP} ] && IP=$( wget -qO- -t1 -T2 ipv4.icanhazip.com )
	[ -z ${IP} ] && IP=$( wget -qO- -t1 -T2 ipinfo.io/ip )
	[ ! -z ${IP} ] && echo ${IP} || echo
	}
	
	Make sure only root can run our script
	rootness(){
	if [[ $EUID -ne 0 ]]; then
	echo "Error:This script must be run as root!" 1>&2
	exit 1
	fi
	}
	
	Check system
	check_sys(){
	local checkType=$1
	local value=$2
	
	local release=''
	local systemPackage=''
	
	if [[ -f /etc/redhat-release ]]; then
	release="centos"
	systemPackage="yum"
	elif cat /etc/issue | grep -q -E -i "debian"; then
	release="debian"
	systemPackage="apt"
	elif cat /etc/issue | grep -q -E -i "ubuntu"; then
	release="ubuntu"
	systemPackage="apt"
	elif cat /etc/issue | grep -q -E -i "centos|red hat|redhat"; then
	release="centos"
	systemPackage="yum"
	elif cat /proc/version | grep -q -E -i "debian"; then
	release="debian"
	systemPackage="apt"
	elif cat /proc/version | grep -q -E -i "ubuntu"; then
	release="ubuntu"
	systemPackage="apt"
	elif cat /proc/version | grep -q -E -i "centos|red hat|redhat"; then
	release="centos"
	systemPackage="yum"
	fi
	
	if [[ ${checkType} == "sysRelease" ]]; then
	if [ "$value" == "$release" ]; then
	return 0
	else
	return 1
	fi
	elif [[ ${checkType} == "packageManager" ]]; then
	if [ "$value" == "$systemPackage" ]; then
	return 0
	else
	return 1
	fi
	fi
	}
	
	Get version
	getversion(){
	if [[ -s /etc/redhat-release ]]; then
	grep -oE  "[0-9.]+" /etc/redhat-release
	else
	grep -oE  "[0-9.]+" /etc/issue
	fi
	}
	
	CentOS version
	centosversion(){
	if check_sys sysRelease centos; then
	local code=$1
	local version="$(getversion)"
	local main_ver=${version%%.*}
	if [ "$main_ver" == "$code" ]; then
	return 0
	else
	return 1
	fi
	else
	return 1
	fi
	}
	
	Disable selinux
	disable_selinux(){
	if [ -s /etc/selinux/config ] && grep 'SELINUX=enforcing' /etc/selinux/config; then
	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
	setenforce 0
	fi
	}
	
	Pre-installation settings
	pre_install(){
	if check_sys packageManager yum || check_sys packageManager apt; then
	Not support CentOS 5
	if centosversion 5; then
	echo "Error: Not supported CentOS 5, please change to CentOS 6+/Debian 7+/Ubuntu 12+ and try again."
	exit 1
	fi
	else
	echo "Error: Your OS is not supported. please change OS to CentOS/Debian/Ubuntu and try again."
	exit 1
	fi
	Set shadowsocks config password
	echo "Please input password for shadowsocks-python:"
	read -p "(Default password: ss):" shadowsockspwd
	[ -z "${shadowsockspwd}" ] && shadowsockspwd="ss"
	echo
	echo "---------------------------"
	echo "password = ${shadowsockspwd}"
	echo "---------------------------"
	echo
	Set shadowsocks config port
	while true
	do
	echo -e "Please input port for shadowsocks-python [1-65535]:"
	read -p "(Default port: 8989):" shadowsocksport
	[ -z "$shadowsocksport" ] && shadowsocksport="8989"
	expr ${shadowsocksport} + 0 &>/dev/null
	if [ $? -eq 0 ]; then
	if [ ${shadowsocksport} -ge 1 ] && [ ${shadowsocksport} -le 65535 ]; then
	echo
	echo "---------------------------"
	echo "port = ${shadowsocksport}"
	echo "---------------------------"
	echo
	break
	else
	echo "Input error! Please input correct numbers."
	fi
	else
	echo "Input error! Please input correct numbers."
	fi
	done
	get_char(){
	SAVEDSTTY=stty -g
	stty -echo
	stty cbreak
	dd if=/dev/tty bs=1 count=1 2> /dev/null
	stty -raw
	stty echo
	stty $SAVEDSTTY
	}
	echo
	echo "Press any key to start...or Press Ctrl+C to cancel"
	char=get_char
	Install necessary dependencies
	if check_sys packageManager yum; then
	yum install -y git wget unzip openssl-devel gcc swig python python-devel python-setuptools autoconf libtool libevent
	yum install -y automake make curl curl-devel zlib-devel perl perl-devel cpio expat-devel gettext-devel
	elif check_sys packageManager apt; then
	apt-get -y update
	apt-get -y install python python-dev python-pip python-setuptools python-m2crypto curl wget unzip gcc swig automake make perl cpio build-essential
	fi
	cd ${cur_dir}
	}
	
	Download files
	download_files(){
	Download libsodium file
	if ! wget --no-check-certificate -O libsodium-1.0.11.tar.gz https://github.com/jedisct1/libsodium/releases/download/1.0.11/libsodium-1.0.11.tar.gz; then
	echo "Failed to download libsodium-1.0.11.tar.gz!"
	exit 1
	fi
	Download Shadowsocks file
	if ! wget --no-check-certificate -O shadowsocks-master.zip https://github.com/shadowsocks/shadowsocks/archive/master.zip; then
	echo "Failed to download shadowsocks python file!"
	exit 1
	fi
	Download Shadowsocks chkconfig file
	if check_sys packageManager yum; then
	if ! wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks -O /etc/init.d/shadowsocks; then
	echo "Failed to download shadowsocks chkconfig file!"
	exit 1
	fi
	elif check_sys packageManager apt; then
	if ! wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-debian -O /etc/init.d/shadowsocks; then
	echo "Failed to download shadowsocks chkconfig file!"
	exit 1
	fi
	fi
	}
	
	Config shadowsocks
	config_shadowsocks(){
	cat > /etc/shadowsocks.json<<-EOF
	{
	"server":"0.0.0.0",
	"server_port":${shadowsocksport},
	"local_address":"127.0.0.1",
	"local_port":1080,
	"password":"${shadowsockspwd}",
	"timeout":300,
	"method":"aes-256-cfb",
	"fast_open":false
	}
	EOF
	}
	
	firewall set
	firewall_set(){
	echo "firewall set start..."
	if centosversion 6; then
	/etc/init.d/iptables status > /dev/null 2>&1
	if [ $? -eq 0 ]; then
	iptables -L -n | grep -i ${shadowsocksport} > /dev/null 2>&1
	if [ $? -ne 0 ]; then
	iptables -I INPUT -m state --state NEW -m tcp -p tcp --dport ${shadowsocksport} -j ACCEPT
	iptables -I INPUT -m state --state NEW -m udp -p udp --dport ${shadowsocksport} -j ACCEPT
	/etc/init.d/iptables save
	/etc/init.d/iptables restart
	else
	echo "port ${shadowsocksport} has been set up."
	fi
	else
	echo "WARNING: iptables looks like shutdown or not installed, please manually set it if necessary."
	fi
	elif centosversion 7; then
	systemctl status firewalld > /dev/null 2>&1
	if [ $? -eq 0 ];then
	firewall-cmd --permanent --zone=public --add-port=${shadowsocksport}/tcp
	firewall-cmd --permanent --zone=public --add-port=${shadowsocksport}/udp
	firewall-cmd --reload
	else
	echo "Firewalld looks like not running, try to start..."
	systemctl start firewalld
	if [ $? -eq 0 ];then
	firewall-cmd --permanent --zone=public --add-port=${shadowsocksport}/tcp
	firewall-cmd --permanent --zone=public --add-port=${shadowsocksport}/udp
	firewall-cmd --reload
	else
	echo "WARNING: Try to start firewalld failed. please enable port ${shadowsocksport} manually if necessary."
	fi
	fi
	fi
	echo "firewall set completed..."
	}
	
	Install Shadowsocks
	install_ss(){
	Install libsodium
	tar zxf libsodium-1.0.11.tar.gz
	cd libsodium-1.0.11/
	./configure && make && make install
	if [ $? -ne 0 ]; then
	echo "libsodium install failed!"
	install_cleanup
	exit 1
	fi
	echo "/usr/local/lib" > /etc/ld.so.conf.d/local.conf
	ldconfig
	Install Shadowsocks
	cd ${cur_dir}
	unzip -q shadowsocks-master.zip
	if [ $? -ne 0 ];then
	echo "unzip shadowsocks-master.zip failed! please check unzip command."
	install_cleanup
	exit 1
	fi
	
	cd ${cur_dir}/shadowsocks-master
	python setup.py install --record /usr/local/shadowsocks_install.log
	
	if [ -f /usr/bin/ssserver ] || [ -f /usr/local/bin/ssserver ]; then
	chmod +x /etc/init.d/shadowsocks
	Add run on system start up
	if check_sys packageManager yum; then
	chkconfig --add shadowsocks
	chkconfig shadowsocks on
	elif check_sys packageManager apt; then
	update-rc.d -f shadowsocks defaults
	fi
	Run shadowsocks in the background
	/etc/init.d/shadowsocks start
	else
	echo
	echo "Shadowsocks install failed! please visit https://teddysun.com/342.html and contact."
	install_cleanup
	exit 1
	fi
	clear
	echo
	echo "Congratulations, shadowsocks server install completed!"
	echo -e "Your Server IP: \033[41;37m $(get_ip) \033[0m"
	echo -e "Your Server Port: \033[41;37m ${shadowsocksport} \033[0m"
	echo -e "Your Password: \033[41;37m ${shadowsockspwd} \033[0m"
	echo -e "Your Local IP: \033[41;37m 127.0.0.1 \033[0m"
	echo -e "Your Local Port: \033[41;37m 1080 \033[0m"
	echo -e "Your Encryption Method: \033[41;37m aes-256-cfb \033[0m"
	echo
	echo "Welcome to visit:https://teddysun.com/342.html"
	echo "Enjoy it!"
	echo
	}
	
	Install cleanup
	install_cleanup(){
	cd ${cur_dir}
	rm -f shadowsocks-master.zip
	rm -rf shadowsocks-master
	rm -f libsodium-1.0.11.tar.gz
	rm -rf libsodium-1.0.11
	}
	
	Uninstall Shadowsocks
	uninstall_shadowsocks(){
	printf "Are you sure uninstall Shadowsocks? (y/n) "
	printf "\n"
	read -p "(Default: n):" answer
	[ -z ${answer} ] && answer="n"
	if [ "${answer}" == "y" ] || [ "${answer}" == "Y" ]; then
	ps -ef | grep -v grep | grep -i "ssserver" > /dev/null 2>&1
	if [ $? -eq 0 ]; then
	/etc/init.d/shadowsocks stop
	fi
	if check_sys packageManager yum; then
	chkconfig --del shadowsocks
	elif check_sys packageManager apt; then
	update-rc.d -f shadowsocks remove
	fi
	delete config file
	rm -f /etc/shadowsocks.json
	rm -f /var/run/shadowsocks.pid
	rm -f /etc/init.d/shadowsocks
	rm -f /var/log/shadowsocks.log
	if [ -f /usr/local/shadowsocks_install.log ]; then
	cat /usr/local/shadowsocks_install.log | xargs rm -rf
	fi
	echo "Shadowsocks uninstall success!"
	else
	echo "uninstall cancelled, nothing to do..."
	fi
	}
	
	Install Shadowsocks-python
	install_shadowsocks(){
	rootness
	disable_selinux
	pre_install
	download_files
	config_shadowsocks
	if check_sys packageManager yum; then
	firewall_set
	fi
	install_ss
	install_cleanup
	}
	
	Initialization step
	action=$1
	[ -z $1 ] && action=install
	case "$action" in
	install)
	install_shadowsocks
	;;
	uninstall)
	uninstall_shadowsocks
	;;
	*)
	echo "Arguments error! [${action} ]"
	echo "Usage: basename $0 {install|uninstall}"
	;;
	esac

[1]:	https://github.com/jekyllcao/shadowsocks-install-script-for-centos-ubuntu-debian
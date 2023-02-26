



## 1.安装httpd过程

```bash
#安装前准备关闭SELinux，和防火墙
#安装wget、bzip2
[root@CentOS8 data]#yum -y install wget bzip2
#下载httpd-2.4.55.tar.bz2
[root@CentOS8 data]#wget https://mirrors.tuna.tsinghua.edu.cn/apache/httpd/httpd-2.4.55.tar.bz2  --no-check-certificate
#习惯将源码编译放到/usr/local/src下
[root@CentOS8 data]#mv httpd-$version.tar.bz2 /usr/local/src/
[root@CentOS8 data]#cd /usr/local/src/
#解压
[root@CentOS8 data]#tar xvf httpd-2.4.55.tar.bz2
[root@CentOS8 data]#cd httpd-2.4.55/
#安装相关包，否则执行make的时候会报错
[root@CentOS8 data]#yum -y install gcc make autoconf apr-devel apr-util-devel pcre-devel openssl-devel redhat-rpm-config
#运行 configure 脚本，生成 Makefile 文件
[root@CentOS8 data]#./configure --prefix=/apps/httpd --sysconfdir=/etc/httpd --enable-ssl
#编译安装
[root@CentOS8 data]#make
#复制文件到相应路径
[root@CentOS8 data]#make install
#二进制程序目录导入至PATH环境变量中
[root@CentOS8 data]#echo 'PATH=/apps/httpd/bin:$PATH' > /etc/profile.d/httpd.sh
#环境变量生效
[root@CentOS8 data]#. /etc/profile.d/httpd.sh
#添加程序所在的用户组
[root@CentOS8 data]#groupadd -g 88 -r apache
#添加运行程序使用的系统用户
[root@CentOS8 data]#useradd -r -u 88 -g apache -s /sbin/nologin -d /var/www/ -c "Apache" apache
#修改配置文件，将程序使用的用户替换成上面添加的apache用户
[root@CentOS8 data]#sed -i -e '/^User/c User apache' -e '/^Group/c Group apache' /etc/httpd/httpd.conf
#启动服务
[root@CentOS8 data]#apachectl start




```



## 2.自动化安装脚本

```bash
#!/bin/bash
#
#*******************************************************
#Author:			xingyuyu
#Date:			    2023-02-24
#Filename:			install_httpd.sh
#Copyright (C):		2023 All rights reserved
#********************************************************
version=2.4.55
CPU=$(lscpu | sed -rn '/^CPU\(s\)/s/.*([0-9]+)/\1/p')
echo -e "\E[1;33m正在检查防火墙状态\E[0m"
systemctl status firewalld.service
if [ $? -eq 0 ];then
    echo -e "\E[1;31m防火墙正在运行当中\E[0m"
    echo -e "\E[1;33m准备关闭火墙\E[0m"
    systemctl stop firewalld.service
    if [ $? -eq 0 ];then
        echo -e "\E[1;32m防火墙关闭Success\E[0m"
        systemctl disable firewalld
        if [ $? -eq 0 ];then
            echo -e "\E[1;32m防火墙卸载Success\E[0m"
        else
            echo -e "\E[1;31m防火墙卸载Faild\E[0m"
            exit
        fi
    else
        echo -e "\E[1;31m防火墙关闭Faild\E[0m"
        exit       
    fi
else
    echo -e "\E[1;31m防火墙已关闭\E[0m"
    systemctl is-enabled firewalld.service &> /dev/null 
    if [ $? -eq 0 ];then
        echo -e "\E[1;32m防火墙卸载Success\E[0m"
    elif [ $? -eq 1 ];then
        echo -e "\E[1;33m防火墙已经卸载，无需再卸载\E[0m"
    else
        echo -e "\E[1;31m防火墙卸载异常\E[0m"
        exit
    fi
fi
selinux=`getenforce`
if [ $selinux == "Enforcing" ];then
    echo -e "\E[1;33m正在关闭SELinux\E[0m"
    sed -ri '/^SELINUX=/s/(.*)enforcing/\1disabled/' /etc/selinux/config
    if [ $? -eq 0 ];then
        echo -e "\E[1;32mSELinux关闭Success\E[0m"
        echo -e "\E[1;32m请重启系统生效！！！\E[0m"
        exit
    else
        echo -e "\E[1;31mSELinux关闭Faild\E[0m"
        exit
    fi
else
    echo -e "\E[1;32mSELinux已是关闭状态\E[0m"
fi
echo -e "\E[1;33m安装wget网络下载工具\E[0m"
yum -y install wget bzip2
echo -e "\E[1;33m准备下载httpd-2.4.55.tar.bz2\E[0m"
`wget https://mirrors.tuna.tsinghua.edu.cn/apache/httpd/httpd-$version.tar.bz2  --no-check-certificate` && { echo -e "\E[1;32mdownload Success\E[0m"; } || { echo -e "\E[1;31mdownload faild\E[0m";exit; }
echo -e "\E[1;33m移动httpd-2.4.55.tar.bz2到/usr/local/src下\E[0m"
mv httpd-$version.tar.bz2 /usr/local/src/
if [ $? -eq 0 ];then
    echo -e "\E[1;32m移动成功\E[0m"
else
    echo -e "\E[1;31m移动失败\E[0m"
    exit 1
fi
echo -e "\E[1;33m进入到/usr/local/src下\E[0m"
cd /usr/local/src/
echo -e "\E[1;33m解压httpd-2.4.55.tar.bz2\E[0m"
tar xvf httpd-$version.tar.bz2
if [ $? -eq 0 ];then
    echo -e "\E[1;32m解压httpd-2.4.55.tar.bz2 Success\E[0m"
else
    echo -e "\E[1;31m解压失败\E[0m"
    exit 1
fi
echo -e "\E[1;33m进入到httpd-2.4.5\E[0m"
cd httpd-$version/
echo -e "\E[1;33m安装gcc等工具\E[0m"
yum -y install gcc make autoconf apr-devel apr-util-devel pcre-devel openssl-devel redhat-rpm-config
if [ $? -eq 0 ];then
    echo -e "\E[1;32m安装gcc工具 Success\E[0m"
else
    echo -e "\E[1;31m安装gcc工具 Failed\E[0m"
    exit 1
fi
echo -e "\E[1;33m执行.configure脚本，生成Makefile文件,httpd.conf文件在/etc/httpd下\E[0m"
./configure --prefix=/apps/httpd --sysconfdir=/etc/httpd --enable-ssl
if [ $? -eq 0 ];then
    echo -e "\E[1;32m执行./configure脚本 Success\E[0m"
else
    echo -e "\E[1;31m执行./configure脚本 Failed\E[0m"
    exit 1
fi
echo -e "\E[1;33m执行make&&make install\E[0m"
make -j $CPU &&make install
if [ $? -eq 0 ];then
    echo -e "\E[1;32m执行make&&make install Success\E[0m"
else
    echo -e "\E[1;31m执行make&&make install Failed\E[0m"
    exit 1
fi
echo -e "\E[1;33m添加环境变量到/etc/profile.d/httpd.sh\E[0m"
echo 'PATH=/apps/httpd/bin:$PATH' > /etc/profile.d/httpd.sh
if [ $? -eq 0 ];then
    echo -e "\E[1;32mPath变量添加 Success\E[0m"
else
    echo -e "\E[1;31mPath变量添加 Success\E[0m"
    exit 1
fi
. /etc/profile.d/httpd.sh
echo -e "\E[1;33m添加apache组\E[0m"
groupadd -g 88 -r apache
if [ $? -eq 0 ];then
    echo -e "\E[1;32mGroup apache added Success\E[0m"
else
    echo -e "\E[1;31mGroup apache added faild\E[0m"
	id apache
	if [ $? -eq 0 ];then
		echo -e "\E[1;31mGroup apache exist\E[0m"
		echo -e "\E[1;31mdelete Group apache\E[0m"
		userdel -rf apache
		groupadd -g 88 -r apache
		echo -e "\E[1;32mGroup apache added again Success\E[0m"
	else
		exit 1
	fi
	
fi
echo -e "\E[1;33m添加apache系统用户\E[0m"
useradd -r -u 88 -g apache -s /sbin/nologin -d /var/www/ -c "Apache" apache
if [ $? -eq 0 ];then
    echo -e "\E[1;32mUser apache added Success\E[0m"
else
    echo -e "\E[1;31mUser apache added faild\E[0m"
    exit 1
fi
echo -e "\E[1;33m修改/etc/httpd/httpd.conf\E[0m"
sed -i -e '/^User/c User apache' -e '/^Group/c Group apache' /etc/httpd/httpd.conf
if [ $? -eq 0 ];then
    echo -e "\E[1;32m修改httpd.conf Success\E[0m"
else
    echo -e "\E[1;31m修改httpd.conf faild\E[0m"
    exit 1
fi
echo -e "\E[1;33m正在启动Apache httpd服务\E[0m"
apachectl start
if [ $? -eq 0 ];then 
    echo -e "\E[1;32mApache httpd服务启动成功\E[0m"
else
    echo -e "\E[1;31mApache httpd服务启动失败\E[0m"
    exit 1
fi

```


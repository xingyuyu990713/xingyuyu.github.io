




[TOC]



> 搭建原理：本次搭建公司内部yum源服务器需要使用挂载的光驱来进行搭建，并且这里使用的autofs自动挂载。

## 1.autofs自动挂载光驱

```shell
[root@centos8 ~]#rpm -q autofs || yum -y install autofs

[root@CentOS8 ~]# systemctl enable --now autofs
Created symlink /etc/systemd/system/multi-user.target.wants/autofs.service → /usr/lib/systemd/system/autofs.service.

[root@CentOS8 ~]# systemctl status autofs


[root@CentOS8 ~]# ls /misc/ 
#发现/misc下面是没有东西的，当我们cd或者ls /misc/cd的时候，就会自动挂载光驱，看到下面的文件。
[root@CentOS8 ~]# ls /misc/cd
CentOS_BuildTag  EFI  EULA  GPL  images  isolinux  LiveOS  Packages  repodata  RPM-GPG-KEY-CentOS-7  RPM-GPG-KEY-CentOS-Testing-7  TRANS.TBL
[root@CentOS8 ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  200G  0 disk
├─sda1   8:1    0  100G  0 part /
├─sda2   8:2    0   50G  0 part /data
├─sda3   8:3    0    2G  0 part [SWAP]
├─sda4   8:4    0    1K  0 part
└─sda5   8:5    0    1G  0 part /boot
sdb      8:16   0   20G  0 disk
sr0     11:0    1  9.5G  0 rom  /misc/cd
sr1     11:1    1 1024M  0 rom
#我们可以看到正常情况下，sr0是挂载到了/misc/cd下的

#如果当我们执行cd /misc/cd 或者 ls /misc/cd 提示"not found"未找到的时候，这个时候不要着急，关机重启就会有了，如果还是没有多重启几次

```

## 2.使用autofs挂载CentOS7光驱

> 因为搭建本地yum服务器的话，需要搭建rpm包(Packages)和原数据(repodata)，同时需要CentOS8和CentOS7两个各自的仓库，BaseOS，APPSteam
>
> 步骤：
>
> 在VMware Workstation pro 17中，对yum服务器右键点击【设置】-【添加】-【CD/DVD驱动器】-【完成】-选择对应的ISO镜像文件，如果你这台是CentOS8的，那么添加的这个就选择CentOS7，反之
>
> 选择完成后一定要点击【确定】

```bash
#此时，添加驱动以后，系统并不会识别
[root@centos8 ~]#lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  200G  0 disk
├─sda1   8:1    0  100G  0 part /
├─sda2   8:2    0   50G  0 part /data
├─sda3   8:3    0    2G  0 part [SWAP]
├─sda4   8:4    0    1K  0 part
└─sda5   8:5    0    1G  0 part /boot
sdb      8:16   0   20G  0 disk
sdc      8:32   0   10G  0 disk
sr0     11:0    1 10.1G  0 rom

#第一块光驱就是sr0，第二块就是sr1，以此类推。可以看到只有一块光驱

#扫描并识别光驱(以下三条命令只适用于scsi硬盘，不适用与Nvme硬盘)
[root@CentOS8 ~]# echo '- - -' /sys/class/scsi_host/host0
[root@CentOS8 ~]# echo '- - -' /sys/class/scsi_host/host1
[root@CentOS8 ~]# echo '- - -' /sys/class/scsi_host/host1

#再次查看光驱是否识别
[root@centos8 ~]#lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  200G  0 disk
├─sda1   8:1    0  100G  0 part /
├─sda2   8:2    0   50G  0 part /data
├─sda3   8:3    0    2G  0 part [SWAP]
├─sda4   8:4    0    1K  0 part
└─sda5   8:5    0    1G  0 part /boot
sdb      8:16   0   20G  0 disk
sdc      8:32   0   10G  0 disk
sr0     11:0    1 10.1G  0 rom
sr1     11:1    1  9.5G  0 rom #sr1光驱就是我们新添加

```

## 3.通过http共享仓库

```bash
#安装httpd
[root@CentOS8 ~]# yum -y install httpd

#自启动http
[root@CentOS8 ~]# systemctl enable --now httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
[root@CentOS8 ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2023-02-11 22:21:52 CST; 2s ago #状态是正常的 active
     Docs: man:httpd.service(8)
 Main PID: 2416 (httpd)
   Status: "Started, listening on: port 80"
    Tasks: 213 (limit: 5802)
   Memory: 25.0M
   CGroup: /system.slice/httpd.service
           ├─2416 /usr/sbin/httpd -DFOREGROUND
           ├─2417 /usr/sbin/httpd -DFOREGROUND
           ├─2418 /usr/sbin/httpd -DFOREGROUND
           ├─2419 /usr/sbin/httpd -DFOREGROUND
           └─2420 /usr/sbin/httpd -DFOREGROUND

2月 11 22:21:48 CentOS8.5 systemd[1]: Starting The Apache HTTP Server...
2月 11 22:21:52 CentOS8.5 systemd[1]: Started The Apache HTTP Server.
2月 11 22:21:55 CentOS8.5 httpd[2416]: Server configured, listening on: port 80
[root@CentOS8 ~]#

#初始化工作(如果不做的话，http服务无法使用)
#关闭防火墙
[root@CentOS8 ~]# systemctl disable --now firewalld
#关闭SElinux
[root@CentOS8 ~]# sed -ri 's/(SELINUX=)disabled/\1disabled/' /etc/selinux/config

#进入存放网页的文件夹
[root@CentOS8 ~]# cd /var/www/html/
#创建Centos7和8的网页文件夹
[root@CentOS8 html]# mkdir -pv centos/{7,8}
#我们仿照镜像网站的路径，例如：
#centos7：https://mirrors.cloud.tencent.com/centos/7/os/x86_64/ 这个路径下面直接是Packages和repodata，因为centos7只要一个仓库
#centos8：https://mirrors.cloud.tencent.com/centos/8/ 这两个路径的下面就是镜像仓库的的地址 centos8下面有BaseOS和APPStream两个仓库，仓库下面才是Packages和repodata

#依次创建所需文件夹
[root@CentOS8 html]# mkdir -pv centos/7/os/x86_64

#sr0就是本系统所使用的的光驱，sr1是后来添加的，例如；如果系统是Centos8的话，那么sr0就是centos8对应的光驱；那么后来添加centos7的光驱，就是sr1
#将sr0挂载到/var/www/html/centos/8 下面
[root@CentOS8 html]# mount /dev/sr0 /var/www/html/centos/8
mount: /var/www/html/centos/8: WARNING: device write-protected, mounted read-only.
#将sr1挂载到/var/www/html/centos/7/os/x86_64 下面
[root@CentOS8 html]# mount /dev/sr1 /var/www/html/centos/7/os/x86_64/

#查看是否挂载成功
[root@CentOS8 html]# tree -d
.
└── centos
    ├── 7
    │   └── os
    │       └── x86_64
    │           ├── EFI
    │           │   └── BOOT
    │           │       └── fonts
    │           ├── images
    │           │   └── pxeboot
    │           ├── isolinux
    │           ├── LiveOS
    │           ├── Packages
    │           └── repodata
    └── 8
        ├── AppStream
        │   ├── Packages
        │   └── repodata
        ├── BaseOS
        │   ├── Packages
        │   └── repodata
        ├── EFI
        │   └── BOOT
        │       └── fonts
        ├── images
        │   └── pxeboot
        └── isolinux

26 directories
[root@CentOS8 html]#
[root@CentOS8 html]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0          11:0    1 10.1G  0 rom  /var/www/html/centos/8  	#centos8挂载
sr1          11:1    1  9.5G  0 rom  /var/www/html/centos/7/os/x86_64	#centos7挂载
nvme0n1     259:0    0  200G  0 disk
├─nvme0n1p1 259:1    0    1G  0 part /boot
├─nvme0n1p2 259:2    0  100G  0 part /
├─nvme0n1p3 259:3    0    2G  0 part [SWAP]
├─nvme0n1p4 259:4    0    1K  0 part
├─nvme0n1p5 259:5    0    1G  0 part /home
└─nvme0n1p6 259:6    0   50G  0 part /data
[root@CentOS8 html]#


```

## 4.访问

web地址：http://10.0.0.109/centos

## 5.给其他服务器配置搭建好的yum仓库

```bash
#给其他centos8服务器配置yum仓库
[root@centos8 html]#vim /etc/yum.repos.d/base.repo
[BaseOs]
name=BaseOS
baseurl=http://10.0.0.109/centos/8/BaseOS/
#baseurl=http://10.0.0.109/centos/$releasever/BaseOS/
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://10.0.0.109/centos/8/AppStream/
#或者baseurl=http://10.0.0.109/centos/$releasever/AppStream/
gpgcheck=0

[extras]
name=extras
baseurl=https://mirrors.aliyun.com/centos/$releasever/extras/$basearch/os
        https://mirrors.cloud.tencent.com/centos/$releasever/extras/$basearch/os/
        https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/os/
gpgcheck=0
enabled=1

[epel]
name=epel
baseurl=https://mirrors.aliyun.com/epel/$releasever/Everything/$basearch/
        https://mirrors.cloud.tencent.com/epel/$releasever/Everything/$basearch/
        https://mirrors.tuna.tsinghua.edu.cn/epel/$releasever/Everything/$basearch/
gpgcheck=0
enabled=1

#查看是否成功
[root@centos8 html]#yum repolist
repo id                                                                                                  repo name
AppStream                                                                                                AppStream
BaseOs                                                                                                   BaseOS
epel                                                                                                     epel
extras                                                                                                   extras

#测试
```

## 6.只有RPM包，没有元数据的情况

>意思就是没有repodata的时候该如何做？

```bash
#模拟在生产环境中自己制作的rpm包
[root@CentOS8 testrepo]# cp /misc/cd/BaseOS/Packages/adcli-0.8.2-12.el8.x86_64.rpm /data/testrepo/
[root@CentOS8 testrepo]# cp /misc/cd/BaseOS/Packages/at-3.1.20-11.el8.x86_64.rpm /data/testrepo/
[root@CentOS8 testrepo]# pwd
/data/testrepo
[root@CentOS8 testrepo]# pwd
/data/testrepo
[root@CentOS8 testrepo]# ls
adcli-0.8.2-12.el8.x86_64.rpm  at-3.1.20-11.el8.x86_64.rpm
[root@CentOS8 testrepo]#

#现在模拟只有rpm，却没有元数据(repodata)
#这种情况只需要创建元数据即可
[root@CentOS8 testrepo]# yum -y install createrepo
[root@CentOS8 testrepo]# createrepo .
Directory walk started
Directory walk done - 2 packages
Temporary output repo path: ./.repodata/
Preparing sqlite DBs
Pool started (with 5 workers)
Pool finished
[root@CentOS8 testrepo]#

[root@CentOS8 testrepo]# ls
adcli-0.8.2-12.el8.x86_64.rpm  at-3.1.20-11.el8.x86_64.rpm  repodata #多出来的文件夹就是元数据
[root@CentOS8 testrepo]#
```

## 7.配置本地epel源

> 将镜像网站上的epel包下载到我们存放网址的文件夹里(/var/www/html)

```sh
#yum所在服务器的配置
[root@CentOS8 yum.repos.d]# vim /etc/yum.repos.d/BaseOS.repo
[BaseOs]
name=BaseOS
baseurl=http://10.0.0.109/centos/$releasever/BaseOS/
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://10.0.0.109/centos/$releasever/AppStream/
gpgcheck=0

[extras]
name=extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/os/
        https://mirrors.aliyun.com/centos/$releasever/extras/$basearch/os
        https://mirrors.cloud.tencent.com/centos/$releasever/extras/$basearch/os/
gpgcheck=0
enabled=1

[epel]
name=epel
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/os/
        https://mirrors.aliyun.com/epel/$releasever/Everything/$basearch/
        https://mirrors.cloud.tencent.com/epel/$releasever/Everything/$basearch/
gpgcheck=0
enabled=1

[root@CentOS8 yum.repos.d]# yum repolist
仓库 id                                                                                                                       仓库名称
AppStream                                                                                                                     AppStream
BaseOs                                                                                                                        BaseOS
epel                                                                                                                          epel
extras                                                                                                                        extras


#CentOS8下载相关仓库包和元数据
[root@CentOS8 ~]# dnf reposync --repoid=epel --download-metadata -p /var/www/html
#--download-metadata 加此选项可以下载元数据
[root@CentOS8 ~]# cd /var/www/html/
[root@CentOS8 html]# ls
centos  epel
[root@CentOS8 html]

#用同样的方法，将CentOS7的epel也下载到epel下面，因为上面使用的epel源中的$releasever会自动获取为8，$basearch会自动获取为x86_64,所以只需要修改里面$releasever改成7会再下载一次即可
[root@CentOS8 html]# dnf reposync --repoid=epel --download-metadata -p /var/www/html/epel/7/x86_64

之后将这些包mv到其他地方，做成自己需要的路径，下面是我自己的路径：
CentOS 8 epel:http://10.0.0.109/epel/8/Everything/x86_64/
CentOS 7 epel:http://10.0.0.109/epel/7/x86_64/
CentOS 8 BaseOS:http://10.0.0.109/centos/8/BaseOS/
CentOS 8 AppStream:http://10.0.0.109/centos/8/AppStream/
CentOS 7:http://10.0.0.109/centos/7/os/x86_64/


#访问网址
http://10.0.0.109/epel/
http://10.0.0.109/centos/
#直接访问http://10.0.0.109是没有的
```

## 8.给其他服务器配置yum源

### CentOS 7

```sh
[base]
name=BaseOS
baseurl=http://10.0.0.109/centos/$releasever/os/$basearch/
gpgcheck=0

[extras]
name=extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/
        https://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
        https://mirrors.cloud.tencent.com/centos/$releasever/extras/$basearch/
gpgcheck=0
enabled=1

[epel]
name=epel
baseurl=http://10.0.0.109/epel/$releasever/$basearch/
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/epel/RPM-GPG-KEY-EPEL-7

```

### CenOS 8

```sh
[BaseOs]
name=BaseOS
baseurl=http://10.0.0.109/centos/$releasever/BaseOS/
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://10.0.0.109/centos/$releasever/AppStream/
gpgcheck=0

[extras]
name=extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/os/
        https://mirrors.aliyun.com/centos/$releasever/extras/$basearch/os
        https://mirrors.cloud.tencent.com/centos/$releasever/extras/$basearch/os/
gpgcheck=0
enabled=1

[epel]
name=epel
baseurl=http://10.0.0.109/epel/$releasever/Everything/$basearch/
gpgcheck=0
enabled=1
```


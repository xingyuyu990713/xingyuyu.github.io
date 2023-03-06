





这里宿主机的型号选择是centos7.9.2009的版本

### 1.宿主机关闭防火墙和selinux，配置ipv4

```sh
#设置SELinux=disabled
vim /etc/selinux/config
SELinux=disabled
查看防火墙状态：firewall-cmd --state
关闭防火墙：systemctl stop firewalld
卸载防火墙：systemctl disable firewalld
vim /etc/sysctl.conf
net.ipv4.ip_forward = 1
```

![image-20230202180627592](https://typora-images-1307361841.cos.ap-beijing.myqcloud.com/img/image-20230202180627592.png)

```sh
重启网络：systemctl restart network && systemctl restart docker
sysctl net.ipv4.ip_forward
```



## 2. 卸载旧版本的docker

```sh
如果之前安装过docker的话需要卸载
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
```



## 3.yum安装docker

```sh
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
如果下载的repo文件不管用，执行：
wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo --no-check-certificate
sed -i 's+download.docker.com+mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo
yum makecache fast
yum -y install docker-ce(默认安装的20.10)
```



## 4.离线安装docker（二进制安装）

```sh
如果无法使用yum源来安装docker的话，那么就必须离线使用tar包进行安装。
下载docker-20.10.7.tgz包，使用wget或者直接下载好，然后上传服务器
wget https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/static/stable/x86_64/docker-20.10.7.tgz 
安装以后会报错
```

![image-20230202181205428](https://typora-images-1307361841.cos.ap-beijing.myqcloud.com/img/image-20230202181205428.png)



```sh
解决：使用无证书检测：yum install -y ca-certificates
再次执行安装便可成功。
```

![image-20230202181320773](https://typora-images-1307361841.cos.ap-beijing.myqcloud.com/img/image-20230202181320773.png)



[详见]: E:\XYY_WorkSpaces\Typora_WorkSpace\二进制安装docker.md

```bash
解压安装包
tar xvf docker-20.10.7.tgz
```

![image-20230206132037483](https://typora-images-1307361841.cos.ap-beijing.myqcloud.com/img/image-20230206132037483.png)

```bash
解压出来这些命令可以使用：./docker/docker version
```

<img src="https://typora-images-1307361841.cos.ap-beijing.myqcloud.com/img/image-20230206132054646.png" alt="image-20230206132054646"  />



```bash
解压出来的这些都是命令，为了保证普通用户也可以执行，将命令放到/usr/bin里
cp docker/* /usr/bin/
创建三个service文：docker.service、dokcer.socket、containerd.service
这三个文件如果是通过yum来安装的话，那么会自动生成，如果是通过二进制tar包来安装的话，这些文件不存在需要手动创建，或者是通过之前yum安装的服务器来拷贝进去。这里使用手动创建
vim /lib/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=infinity
# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
启动：systemctl enable containerd &systemctl start containerd &systemctl status containerd
增加docker组，否则docker无法启动：groupadd docker

```

```bash
vim /lib/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target docker.socket firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket containerd.service

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always

# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3

# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not support it.
# Only systemd 226 and above support this option.
TasksMax=infinity

# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes

# kill only the docker process, not all processes in the cgroup
KillMode=process
OOMScoreAdjust=-500

[Install]
WantedBy=multi-user.target
启动：systemctl enable docker.service &systemctl start docker.service &systemctl status docker.service

```

```sh
vim /lib/systemd/system/docker.socke
[Unit]
Description=Docker Socket for the API

[Socket]
ListenStream=/var/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target

```

```bash
systemctl enable docker.socket &systemctl start docker.socket &systemctl status docker.socket

systemctl restart docker

docker info 
docker version

```



## 5.安装并且启动docker

```bash
配置完成镜像源，开始安装docker，这里安装的最新版的docker
yum install -y docker-ce
启动docker:
systemcetl start docker
自启动docker：
systemctl enable docker
查看docker的版本:
docker version
查看docker的基本信息:
docker info
查看docker当前的运行状态
systemctl status docker

```



## 6.docker拉取镜像

```bash
使用docker拉取centos7.9.2009系统作为我们镜像的系统:
docker pull centos:centos7.9.2009

查看docker拉取的镜像:
docker images

查看docker开启了哪些容器:
docker ps -a

```



## 7.运行docker容器

```bash
将拉取的centos7.9.2009的镜像启动，将其作为一个容器
docker run -itd --privileged -p 2222:22 -p 80:81 -p 260:443 -p 261:3306 -p 262:3307 -p 6369:6379 -v /sys/fs/cgroup:/sys/fs/cgroup --name keshihua xingyuyu123/hzky:keshihua  /usr/sbin/init

启动api:
docker run -itd --privileged -p 2222:22 -p 80:80 -p 2181:2181 -p 9092:9092 -p 8983:8983 -p 8080:8080 -v /sys/fs/cgroup:/sys/fs/cgroup --name api centos:centos7.9.2009  /usr/sbin/init

启动容器时，映射cgroup内核限制资源目录，即-v /sys/fs/cgroup:/sys/fs/cgroup,否则mysql起不来（如果有需要安装mysql需要这个，为了方便直接加上不影响）
前面的端口是宿主机的端口，指的是宿主机的端口映射到容器中端口是多少

-p 2222:22:增加端口映射，前面的映射到宿主机的端口，后面的22是容器内部真正的端口，以此类推
--name：api，是自定义的名字，centos:centos7.9.2009这个是拉取镜像的名称和对应的版本号

```

```bash
如果执行上述命令，提示如下错误：
WARNING: IPv4 forwarding is disabled. Networking will not work.

vim /etc/sysctl.conf
添加一条：net.ipv4.ip_forward=1
#重启network服务
systemctl restart network && systemctl restart docker
#查看是否修改成功 （备注：返回1，就是成功）
sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
删除重新创建容器
启动容器以后，可以通过iptables来查看当前的规则
iptables -t nat -vnL
结果如下：
tcp dpt:2222 to:172.17.0.2:22 前面的是宿主机的端口，后面的是容器的端口
```



## 8.通过xshell等终端工具连接docker容器

```bash
启动docker，进入docker容器，如果是守护态容器，可以通过下面的方式进入：
先查看当前运行的容器状态，获取当前运行容器的CONTAINER ID
[root@centos7 ~]#docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

进入容器
[root@centos7 ~]#docker exec -it CONTAINER ID/name /bin/bash  #这里的CONTAINER ID对应的就是容器的id或者可以用给他启的名字name来启动

使用passwd密码来修改密码（如提示没有这个命令行使用yum install passwd安装）
passwd
xxx密码
xxx确认密码
这里设置的密码是:root


安装Openssh（docker 容器中执行）
安装ssh: yum -y install openssh-server
安装openssh-clients：yum -y install openssh-clients

修改SSH配置文件以下选项，去掉#注释，将四个选项启用：
vi /etc/ssh/sshd_config 如果下方没有的，就在最后面添加项
RSAAuthentication yes #启用 RSA 认证（这个没有需要添加）
PubkeyAuthentication yes #启用公钥私钥配对认证方式
AuthorizedKeysFile      .ssh/authorized_keys #公钥文件路径（和上面生成的文件同）
PermitRootLogin yes #root能使用ssh登录


重启ssh服务，并设置开机启动：
安装server：yum install initscripts -y
service sshd restart
或者
systemctl restart sshd
systemctl enable sshd.service
退出容器并保存更改
使用exit命令或者ctrl+C来退出当前运行的容器：
此时使用xshell连接docker容器
ip: 为宿主主机的ip，而不是docker容器的ip
端口号：2222 
用户名： root
密码： 上面password部分设置的密码 centos7web

一定要关闭容器的防火墙或者防火墙透出端口，否则外界无法连通容器的端口
yum -y install firewalld
yum -y install net-tools
查看当前防火墙的状态: firewall-cmd --state
如果是running的话，那么首先关闭防火墙
systemctl stop firewalld
关闭防火墙以后，确认防火墙是否是自启动
systemctl is-enabled firewalld
enabled表示是自启动，那么将防火墙卸载，关闭自启动
systemctl disable firewalld
同理，宿主机也要卸载防火墙
```



## 9.docker修改时区

```bash
1、	查看当前时区：timedatectl
[root@centos7 ~]#timedatectl 
      Local time: 一 2023-02-06 14:09:20 CST
  Universal time: 一 2023-02-06 06:09:20 UTC
        RTC time: 一 2023-02-06 06:09:20
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
[root@centos7 ~]

注意：系统时区是通过符号链接 /etc/localtime 到目录中的二进制时区标识符来 配置的/usr/share/zoneinfo 。ls检查时区的另一个选项是使用以下命令显示符号链接指向的路径 ：
ls -l /etc/localtime

2. 在 CentOS 中更改时区 
在更改时区之前，您需要找出要使用的时区的长名称。时区使用“地区/城市”格式。要列出所有可用时区，请使用以下选项调用 timedatectl 命令 ：
timedatectl list-timezones
修改时区 sudo timedatectl set-timezone "Asia/Shanghai"
查看当前时间、时区
date -R
timedatectl

执行完成后再运行timedatectl命令查看即可【记得重启后-docker同步才会生效】
reboot

```



## 10.增加tab代码提示

```bash
#安装Linux命令补全工具
yum -y install bash-completion
#执行bash或者reboot重启系统
bash
#如果上述的命令执行了有问题可以执行功能下面的命令
yum install epel-release -y
yum install bash-completion bash-completion-extras -y

#Ubuntu也可以进行命令补全
apt install -y bash-completion

```



## 11.增加端口映射

### 方法一：修改配置文件

```bash
问题:如果在创建容器的时候，没加上固定的映射端口，怎么能重新加呢?
这里有两种解决方式
方法一:修改配置文件
先这个容器停掉，然后再停止docker，最后找到容器配置端口的文件，增加端口就可以了，然后重新启动即可.
1.	查看当前有哪个docker容器正在运行当中 docker ps
 
2.	停止该容器 docker stop web或者docker stop 10dc410fb6d4
停止docker服务:
systemctl stop docker.socket
systemctl stop docker.service
systemctl stop container.service
systemctl stop docker

3.	进入到docker容器的目录，找到对应编号的容器目录
cd /var/lib/docker/containers/
找到对应容器的id，cd，docker ps 显示的container id是选取了一部分，真正的container id很长
需要修改config.v2.json和hostconfig.json这两个文件
```

```json
1.config.v2.json
将文件的内容通过json工具格式化一下就看的清楚了
在” ExposedPorts”下添加容器内部自己的端口
"8080/tcp": {}

{
  "StreamConfig": {},
  "State": {
    "Running": true,
    "Paused": false,
    "Restarting": false,
    "OOMKilled": false,
    "RemovalInProgress": false,
    "Dead": false,
    "Pid": 3553,
    "ExitCode": 0,
    "Error": "",
    "StartedAt": "2023-02-06T06:24:09.173425372Z",
    "FinishedAt": "0001-01-01T00:00:00Z",
    "Health": null
  },
  "ID": "0592650f92d18b159be22c5d2539a8e7d4427d0605d3a5dba11f336753cd7fc3",
  "Created": "2023-02-06T06:24:08.289114604Z",
  "Managed": false,
  "Path": "/usr/sbin/init",
  "Args": [],
  "Config": {
    "Hostname": "0592650f92d1",
    "Domainname": "",
    "User": "",
    "AttachStdin": false,
    "AttachStdout": false,
    "AttachStderr": false,
    "ExposedPorts": {
      "2181/tcp": {},
      "22/tcp": {},
      "80/tcp": {},
      "8080/tcp": {},
      "8983/tcp": {},
      "9092/tcp": {}
    },
    "Tty": true,
    "OpenStdin": true,
    "StdinOnce": false,
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ],
    "Cmd": [
      "/usr/sbin/init"
    ],
    "Image": "centos:centos7.9.2009",
    "Volumes": null,
    "WorkingDir": "",
    "Entrypoint": null,
    "OnBuild": null,
    "Labels": {
      "org.label-schema.build-date": "20201113",
      "org.label-schema.license": "GPLv2",
      "org.label-schema.name": "CentOS Base Image",
      "org.label-schema.schema-version": "1.0",
      "org.label-schema.vendor": "CentOS",
      "org.opencontainers.image.created": "2020-11-13 00:00:00+00:00",
      "org.opencontainers.image.licenses": "GPL-2.0-only",
      "org.opencontainers.image.title": "CentOS Base Image",
      "org.opencontainers.image.vendor": "CentOS"
    }
  },
  "Image": "sha256:eeb6ee3f44bd0b5103bb561b4c16bcb82328cfe5809ab675bb17ab3a16c517c9",
  "NetworkSettings": {
    "Bridge": "",
    "SandboxID": "1d563c887ac4d7f83f72511066fa17e5b27bc3652ce9e608065ed4a04ccc5d1d",
    "HairpinMode": false,
    "LinkLocalIPv6Address": "",
    "LinkLocalIPv6PrefixLen": 0,
    "Networks": {
      "bridge": {
        "IPAMConfig": null,
        "Links": null,
        "Aliases": null,
        "NetworkID": "797f847018a69e2d81ba9f63bffa536216221cfabb688c4f76e9f88f48255d51",
        "EndpointID": "53259c119cb19d3763abe0485645198c0b80a762cc1d72279d0d0f8742b04712",
        "Gateway": "172.17.0.1",
        "IPAddress": "172.17.0.2",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "MacAddress": "02:42:ac:11:00:02",
        "DriverOpts": null,
        "IPAMOperational": false
      }
    },
    "Service": null,
    "Ports": {
      "2181/tcp": [
        {
          "HostIp": "0.0.0.0",
          "HostPort": "2181"
        },
        {
          "HostIp": "::",
          "HostPort": "2181"
        }
      ],
      "22/tcp": [
        {
          "HostIp": "0.0.0.0",
          "HostPort": "2222"
        },
        {
          "HostIp": "::",
          "HostPort": "2222"
        }
      ],
      "80/tcp": [
        {
          "HostIp": "0.0.0.0",
          "HostPort": "80"
        },
        {
          "HostIp": "::",
          "HostPort": "80"
        }
      ],
      "8080/tcp": [
        {
          "HostIp": "0.0.0.0",
          "HostPort": "8080"
        },
        {
          "HostIp": "::",
          "HostPort": "8080"
        }
      ],
      "8983/tcp": [
        {
          "HostIp": "0.0.0.0",
          "HostPort": "8983"
        },
        {
          "HostIp": "::",
          "HostPort": "8983"
        }
      ],
      "9092/tcp": [
        {
          "HostIp": "0.0.0.0",
          "HostPort": "9092"
        },
        {
          "HostIp": "::",
          "HostPort": "9092"
        }
      ]
    },
    "SandboxKey": "/var/run/docker/netns/1d563c887ac4",
    "SecondaryIPAddresses": null,
    "SecondaryIPv6Addresses": null,
    "IsAnonymousEndpoint": false,
    "HasSwarmEndpoint": false
  },
  "LogPath": "/var/lib/docker/containers/0592650f92d18b159be22c5d2539a8e7d4427d0605d3a5dba11f336753cd7fc3/0592650f92d18b159be22c5d2539a8e7d4427d0605d3a5dba11f336753cd7fc3-json.log",
  "Name": "/api",
  "Driver": "overlay2",
  "OS": "linux",
  "MountLabel": "",
  "ProcessLabel": "",
  "RestartCount": 0,
  "HasBeenStartedBefore": true,
  "HasBeenManuallyStopped": false,
  "MountPoints": {
    "/sys/fs/cgroup": {
      "Source": "/sys/fs/cgroup",
      "Destination": "/sys/fs/cgroup",
      "RW": true,
      "Name": "",
      "Driver": "",
      "Type": "bind",
      "Propagation": "rprivate",
      "Spec": {
        "Type": "bind",
        "Source": "/sys/fs/cgroup",
        "Target": "/sys/fs/cgroup"
      },
      "SkipMountpointCreation": false
    }
  },
  "SecretReferences": null,
  "ConfigReferences": null,
  "AppArmorProfile": "",
  "HostnamePath": "/var/lib/docker/containers/0592650f92d18b159be22c5d2539a8e7d4427d0605d3a5dba11f336753cd7fc3/hostname",
  "HostsPath": "/var/lib/docker/containers/0592650f92d18b159be22c5d2539a8e7d4427d0605d3a5dba11f336753cd7fc3/hosts",
  "ShmPath": "",
  "ResolvConfPath": "/var/lib/docker/containers/0592650f92d18b159be22c5d2539a8e7d4427d0605d3a5dba11f336753cd7fc3/resolv.conf",
  "SeccompProfile": "",
  "NoNewPrivileges": false,
  "LocalLogCacheMeta": {
    "HaveNotifyEnabled": false
  }
}
```

```json
2.hostconfig.json

    "8080/tcp": [
        {
          "HostIp": "",
          "HostPort": "8888"
        }
      ]
意思是容器中的8080映射到宿主机中的ip是8088，修改完以后使用工具将格式压缩回之前的格式，然后替换配置文件中的.

{
  "Binds": [
    "/sys/fs/cgroup:/sys/fs/cgroup"
  ],
  "ContainerIDFile": "",
  "LogConfig": {
    "Type": "json-file",
    "Config": {}
  },
  "NetworkMode": "default",
  "PortBindings": {
    "2181/tcp": [
      {
        "HostIp": "",
        "HostPort": "2181"
      }
    ],
    "22/tcp": [
      {
        "HostIp": "",
        "HostPort": "2222"
      }
    ],
    "80/tcp": [
      {
        "HostIp": "",
        "HostPort": "80"
      }
    ],
    "8080/tcp": [
      {
        "HostIp": "",
        "HostPort": "8080"
      }
    ],
    "8983/tcp": [
      {
        "HostIp": "",
        "HostPort": "8983"
      }
    ],
    "9092/tcp": [
      {
        "HostIp": "",
        "HostPort": "9092"
      }
    ]
  },
  "RestartPolicy": {
    "Name": "no",
    "MaximumRetryCount": 0
  },
  "AutoRemove": false,
  "VolumeDriver": "",
  "VolumesFrom": null,
  "CapAdd": null,
  "CapDrop": null,
  "CgroupnsMode": "host",
  "Dns": [],
  "DnsOptions": [],
  "DnsSearch": [],
  "ExtraHosts": null,
  "GroupAdd": null,
  "IpcMode": "private",
  "Cgroup": "",
  "Links": null,
  "OomScoreAdj": 0,
  "PidMode": "",
  "Privileged": true,
  "PublishAllPorts": false,
  "ReadonlyRootfs": false,
  "SecurityOpt": [
    "label=disable"
  ],
  "UTSMode": "",
  "UsernsMode": "",
  "ShmSize": 67108864,
  "Runtime": "runc",
  "ConsoleSize": [
    0,
    0
  ],
  "Isolation": "",
  "CpuShares": 0,
  "Memory": 0,
  "NanoCpus": 0,
  "CgroupParent": "",
  "BlkioWeight": 0,
  "BlkioWeightDevice": [],
  "BlkioDeviceReadBps": null,
  "BlkioDeviceWriteBps": null,
  "BlkioDeviceReadIOps": null,
  "BlkioDeviceWriteIOps": null,
  "CpuPeriod": 0,
  "CpuQuota": 0,
  "CpuRealtimePeriod": 0,
  "CpuRealtimeRuntime": 0,
  "CpusetCpus": "",
  "CpusetMems": "",
  "Devices": [],
  "DeviceCgroupRules": null,
  "DeviceRequests": null,
  "KernelMemory": 0,
  "KernelMemoryTCP": 0,
  "MemoryReservation": 0,
  "MemorySwap": 0,
  "MemorySwappiness": null,
  "OomKillDisable": false,
  "PidsLimit": null,
  "Ulimits": null,
  "CpuCount": 0,
  "CpuPercent": 0,
  "IOMaximumIOps": 0,
  "IOMaximumBandwidth": 0,
  "MaskedPaths": null,
  "ReadonlyPaths": null
}
```

### 方法二：打包重启运行容器

```shell
在容器内部部署完成所有的工作以后，将这个容器打包成镜像，这里的镜像就是一个tar包，重新加载一下这个镜像，在加载的时候像上面一样添加端口映射，以8080端口为例:
docker run -itd --privileged -p 2222:22 -p 80:81 -p 260:443 -p 261:3306 -p 262:3307 -p 6369:6379 -p 8888:8080 -v /sys/fs/cgroup:/sys/fs/cgroup --name web centos:centos7.9.2009  /usr/sbin/init

查看当前容器所有端口映射情况:
docker port web

```



## 12.docker自启动，container自启动

```shell
安装好docker，需要让docker自启动
查看docker是否是自启动：
systemctl list-unit-files | grep docker

如果是通过yum来安装的话，当时只执行了systemctl enable docker这个命令，那么这里面的三个服务只有一个自启动的，所以我们现在让三个都自启。
systemctl enable docker.service
systemctl enable docker.socket
systemctl enable containerd.service

宿主机当中的docker自启成功了，下面让我们服务所在的容器自启动。
在docker启动容器可以增加参数来达到,当docker 服务重启之后 自动启动容器.（这样会新创建一个容器，在新建容器的时候可以这样创建，如果这个容器已经创建好了就不能这样了，只能使用update）
docker run --restart=always 容器名称或容器id
如果容器已经启动,可以通过update命令进行修改.
docker update --restart=always 容器名称或容器id
如果你想取消掉，命令如下:
docker update --restart=no 容器名称或容器id
这里应该启动api容器：
docker update --restart=always api
## 取消全部容器
docker update --restart=no $(docker ps -aq)

```



## 13.打包容器生成镜像并运行

### 13.1 提交容器，上传到docker hub

```bash
这里以应用可视化为例：通过将容器生成镜像，然后上传到docker仓库。
首先停止容器：docker stop 容器名
docker   commit -m "描述信息" -a "作者" 容器id 目标镜像名:[TAG]
docker commit -m "hello" -a "hzky" web keshihua:1.0
```

![image-20230206145450143](https://typora-images-1307361841.cos.ap-beijing.myqcloud.com/img/image-20230206145450143.png)

```
通过commit提交以后，如图：
```

![image-20230206145528993](https://typora-images-1307361841.cos.ap-beijing.myqcloud.com/img/image-20230206145528993.png)

```bash
-a :提交的镜像作者；
-c :使用Dockerfile指令来创建镜像；
-m :提交时的说明文字；
-p :在commit时，将容器暂停
web是要打包的容器
keshihua:1.0是新的镜像名和版本号
提交的信息可以通过：docker inspect 镜像的名称或者镜像的id
可以查看历史信息：docker history 镜像的名称或者镜像的id
登录docker仓库：docker login
用户名：xingyuyu123
密码：xingyuyu123
docker tag keshihua:1.0 xingyuyu123/hzky:keshihua
keshihua:1.0 是要打包的镜像的名和版本号，后面的是打成什么
docker tag 是为了将镜像名打成docker hub的仓库名字，用tag名字不一样来区分
执行完以后，如图所示：
```

![image-20230206145610632](https://typora-images-1307361841.cos.ap-beijing.myqcloud.com/img/image-20230206145610632.png)

这是为了和我docker hub中仓库的名称对应起来，这样才可以找到

```bash
docker hub仓库地址：
https://hub.docker.com/repository/docker/xingyuyu123/hzky

这里是将容器提交生成镜像，然后将镜像打一个tag，xingyuyu123是docker hub的用户名，hzky是docker hub创建的仓库名，keshihua是命名的版本号，前面的keshihua:1.0是commit提交自定义命名的repository和tag

上传到docker hub仓库
docker push xingyuyu123/hzky:keshihua
```

![image-20230206145812023](https://typora-images-1307361841.cos.ap-beijing.myqcloud.com/img/image-20230206145812023.png)

到docker hub查看一下，是否上传成功

![image-20230206145837735](https://typora-images-1307361841.cos.ap-beijing.myqcloud.com/img/image-20230206145837735.png)

上传成功！！！

### 13.2 从docker hub拉取镜像

```
在其他服务器上首先安装docker，通过yum来安装或者通过二进制安装，启动，设置自启。
登录docker hub仓库，因为这里的仓库设置的私有的
docker login
xingyuyu123
xingyuyu123
docker pull xingyuyu123/hzky:keshihua
这里一定要和docker hub里的对应起来
```

![image-20230206145950398](https://typora-images-1307361841.cos.ap-beijing.myqcloud.com/img/image-20230206145950398.png)

然后通过docker images查看是否拉取成功



### 13.3   运行拉取成功的镜像

```bash
docker run -itd --privileged -p 2222:22 -p 81:81  -p 3306:3306  -p 6369:6379 -v /sys/fs/cgroup:/sys/fs/cgroup --name ksh xingyuyu123/hzky:keshihua  /usr/sbin/init
```



## 14.打包容器生成tar包并运行

```bash
这里使用的是将容器直接打成tar包，然后移植到其他服务器直接加载就可以使用了
方式一：使用export导出快照的方式
将容器到出为tar包，这里导出的相当于是一个快照，容量较小。
将容器停止 docker stop 容器的id和容器的名称
导出tar包 docker export -o /opt web > keshihua.tar
-o 可以加具体生成tar包到哪个路径

将tar包发送到其他服务器
scp -r keshihua.tar 192.168.124.39:/root/

导入tar包，生成镜像
docker import keshihua.tar keshihua:hzky
keshihua:hzky 这个是自定义命名

也可以使用 cat keshihua.tar | docker import - keshihua.tar
这样会使用默认的一些参数，不能自定义

启动：docker run -itd --privileged -p 2222:22 -p 80:81 -p 260:443 -p 261:3306 -p 262:3307 -p 6369:6379 -v /sys/fs/cgroup:/sys/fs/cgroup --name centos7web keshihua.tar:latest  /usr/sbin/init
方式二：
方式一打包只是一个快照，export命令是从容器（container）中导出tar文件，而save命令则是从镜像（images）中导出
文件大小不同：export 导出的镜像文件体积小于 save 保存的镜像
是否可以对镜像重命名
 docker import 可以为镜像指定新名称
 docker load 不能对载入的镜像重命名

是否可以同时将多个镜像打包到一个文件中
docker export 不支持
docker save 支持

是否包含镜像历史
export 导出（import 导入）是根据容器拿到的镜像，再导入时会丢失镜像所有的历史记录和元数据信息（即仅保存容器当时的快照状态），所以无法进行回滚操作
而 save 保存（load 加载）的镜像，没有丢失镜像的历史，可以回滚到之前的层（layer）
 
应用场景不同
docker export 的应用场景：主要用来制作基础镜像，比如我们从一个 ubuntu 镜像启动一个容器，然后安装一些软件和进行一些设置后，使用 docker export 保存为一个基础镜像。然后，把这个镜像分发给其他人使用，比如作为基础的开发环境。
docker save 的应用场景：如果我们的应用是使用 docker-compose.yml 编排的多个镜像组合，但我们要部署的客户服务器并不能连外网。这时就可以使用 docker save 将用到的镜像打个包，然后拷贝到客户服务器上使用 docker load 载入。
首先，提交容器生成镜像，对于镜像来说，如果不显式地指定tag,则默认会选择latest标签
docker commit web keshihua

通过save打包（打包的时候一定要对应起来，通过repository和tag来指定镜像）
docker save -o /root/hzky_ksh.tar keshihua:latest
发送到其他服务器，然后加载
scp hzky_ksh.tar 192.168.124.39:/root/
使用docker load命令载入镜像
docker load -i hzky_ksh.tar
这个加载以后，为什么镜像默认叫keshihua，这是因为commit提交的时候就叫keshihua
使用docker run运行镜像生成容器（启动的时候可以设置容器的别人）
docker run -itd --privileged -p 2222:22 -p 80:81 -p 260:443 -p 261:3306 -p 262:3307 -p 6369:6379 -v /sys/fs/cgroup:/sys/fs/cgroup --name centos7web keshihua:latest  /usr/sbin/init

```

## 15.容器的一些基本命令

```BASH
部署api所有的组件都是在容器中进行的，最后将容器打包生成镜像.
查看所有容器：docker ps -a
查看已经启动的服务：systemctl list-units --type=service
查看是否设置开机启动：systemctl list-unit-files | grep docker
设置开机启动：systemctl enable docker.service
取消开机启动，执行命令：systemctl disable docker.service
已经启动的容器设置容器自启：docker update --restart=always 容器名称或容器id
没有启动的容器设置开机自启：docker run --restart=always 容器名称或容器id
重命名镜像：docker tag 镜像id 新的repository:tag

重命名容器:docker rename old_name new_name
docker拉取命令：docker pull 
systemctl is-enabled docker.service  检查服务是否开机启动
systemctl is-enabled docker.socket
systemctl is-enabled containerd.service
停止容器：docker stop 容器名
删除容器:docker container rm 容器名称或者容器id
删除拉取过的镜像：docker rmi -f c52f59215119
也可以 docker rmi -f REPOSITORY: TAG
例如：docker rmi -f xingyuyu123/centos:centos7.9.2009
宿主机查看容器开启哪些映射端口：docker port api
查看docker版本：docker version
查看docker信息：docker info
查看docker有哪些镜像：docker images
```

## 16.docker 打包api

```sh
docker commit -m "API接口治理" -a "hzky" api api:api
查看docker 镜像的描述信息、作者信息以及端口映射信息等。
docker inspect api:api

docker history api:api
打tag 
docker tag api:api xingyuyu123/hzky:api

上传docker hub仓库
docker push xingyuyu123/hzky:api

这里的报错提示是由于没有登录：docker login
docker push xingyuyu123/hzky:api
本地保存一份：
docker save -o /root/api.tar api:api

```

17.docker运行api镜像

```sh
docker load -i api.tar
docker run -itd --privileged -p 2222:22 -p 80:80 -p 2181:2181 -p 9092:9092 -p 8983:8983 -p 8080:8080 -v /sys/fs/cgroup:/sys/fs/cgroup --name api api:api  /usr/sbin/init

```


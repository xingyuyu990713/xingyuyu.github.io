




# 配置YUM源

阿里云提供了写好的CentOS和Ubuntu的仓库文件，下载链接如下：

```sh
http://mirrors.aliyun.com/repo/
```

各大镜像源网址

```bash
#阿里云 
https://mirrors.aliyun.com/

#腾讯软件源
https://mirrors.cloud.tencent.com/

#华为云
https://mirrors.huaweicloud.com/

#清华大学开源软件镜像站
https://mirrors.tuna.tsinghua.edu.cn/

#网易开源镜像站
https://mirrors.163.com/

#CentOS官网
https://vault.centos.org/

#公云
https://mirrors.pubyun.com/

#搜狐
http://mirrors.sohu.com/

#首都在线
http://mirrors.yun-idc.com/

https://wiki.centos.org/

#Ubuntu官网
https://cn.ubuntu.com/

#中国科技大学
https://mirrors.ustc.edu.cn/

#北京外国语大学
https://mirrors.bfsu.edu.cn

#中国科学技术大学
https://mirrors.ustc.edu.cn/

#北京交通大学
https://mirror.bjtu.edu.cn/

#上海交通大学
https://ftp.sjtu.edu.cn/

#北京理工大学
https://mirror.bit.edu.cn/

#浙江大学
https://mirrors.zju.edu.cn/

#华中科技大学
http://mirrors.hust.edu.cn/

#东北大学
http://mirror.neu.edu.cn

#哈尔滨工业大学
http://mirrors.hit.edu.cn/

#大连理工大学
http://mirror.dlut.edu.cn/

#大连东软信息学院
https://mirrors.neusoft.edu.cn/

#南京大学
http://mirrors.nju.edu.cn/

#南京邮电大学
https://mirrors.njupt.edu.cn/

#兰州大学
http://mirror.lzu.edu.cn/

#重庆大学
http://mirrors.cqu.edu.cn/

```

## CentOS 7 配置yum源

```sh
[root@centos7 yum.repos.d]#cd /etc/yum.repos.d/ #进入到对应的目录下
[root@centos7 yum.repos.d]#touch Base.repo #创建一个以.repo结尾的文件
```

将下面文件内容copy到Base.repo

```sh
[base]
name=BaseOS
baseurl=https://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        https://mirrors.cloud.tencent.com/centos/$releasever/os/$basearch/
        https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
gpgcheck=0

[extras]
name=extras
baseurl=https://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
        https://mirrors.cloud.tencent.com/centos/$releasever/extras/$basearch/
        https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/
gpgcheck=0
enabled=1

[epel]
name=epel
baseurl=https://mirrors.aliyun.com/epel/$releasever/$basearch/
        https://mirrors.cloud.tencent.com/epel/$releasever/$basearch/
        https://mirrors.tuna.tsinghua.edu.cn/epel/$releasever/$basearch/
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/epel/RPM-GPG-KEY-EPEL-7


#上面的gpgcheck=1 #安装包前要做包的合法和完整性校验
#如果是写1的话，那么就要配置gpgkey，例如：https://mirrors.tuna.tsinghua.edu.cn/centos/ 找到RPM-GPG-KEY-CentOS-7复制地址
```

```sh
#查看yum源
[root@centos7 yum.repos.d]#yum repolist
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base:
 * epel: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
源标识                                                                                                          源名称                                                                                                  状态
base/7/x86_64                                                                                                   BaseOS                                                                                                   4,070
epel/7/x86_64                                                                                                   epel                                                                                                    13,744
extras/7/x86_64                                                                                                 extras                                                                                                     515
repolist: 18,329
[root@centos7 yum.repos.d]#

```

## CentOS 8 配置yum源

```bash
 #进入到对应的目录下
[root@centos7 yum.repos.d]#cd /etc/yum.repos.d/

#创建一个以.repo结尾的文件
[root@centos7 yum.repos.d]#touch Base.repo
```

将下面文件内容copy到Base.repo

```bash
[xyyBaseOS]
name=BaseOS
baseurl=https://mirrors.aliyun.com/centos/$releasever/BaseOS/$basearch/os/
        https://mirrors.cloud.tencent.com/centos/$releasever/BaseOS/$basearch/os/
        https://mirrors.tuna.tsinghua.edu.cn/cc/$releasever/BaseOS/$basearch/os/
gpgcheck=0

[xyyAppStream]
name=AppStream
baseurl=https://mirrors.aliyun.com/centos/$releasever/AppStream/$basearch/os/
        https://mirrors.cloud.tencent.com/centos/$releasever/AppStream/$basearch/os/
        https://mirrors.tuna.tsinghua.edu.cn/cc/$releasever/AppStream/$basearch/os/
gpgcheck=0

[xyyextras]
name=extras
baseurl=https://mirrors.aliyun.com/centos/$releasever/extras/$basearch/os
        https://mirrors.cloud.tencent.com/centos/$releasever/extras/$basearch/os/
        https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/os/
gpgcheck=0
enabled=1

[xyyepel]
name=epel
baseurl=https://mirrors.aliyun.com/epel/$releasever/Everything/$basearch/
        https://mirrors.cloud.tencent.com/epel/$releasever/Everything/$basearch/
        https://mirrors.tuna.tsinghua.edu.cn/epel/$releasever/Everything/$basearch/
gpgcheck=0
enabled=1



```

```sh
[root@centos8 yum.repos.d]#yum repolist
repo id                                                                                                   repo name
xyyAppStream                                                                                              AppStream
xyyBaseOS                                                                                                 BaseOS
xyyepel                                                                                                   epel
xyyextras                                                                                                 extras
[root@centos8 yum.repos.d]#

#查看详细信息
[root@centos8 yum.repos.d]#yum repolist -v
Loaded plugins: builddep, changelog, config-manager, copr, debug, debuginfo-install, download, generate_completion_cache, groups-manager, needs-restarting, playground, repoclosure, repodiff, repograph, repomanage, reposync
YUM version: 4.7.0
cachedir: /var/cache/dnf
Last metadata expiration check: 1:04:59 ago on Tue 07 Feb 2023 10:48:29 PM CST.
Repo-id            : xyyAppStream
Repo-name          : AppStream
Repo-revision      : 8.5.2111
Repo-distro-tags      : [cpe:/o:centos:centos:8]:  , 8, C, O, S, e, n, t
Repo-updated       : Sat 13 Nov 2021 09:02:30 AM CST
Repo-pkgs          : 6,165
Repo-available-pkgs: 5,281
Repo-size          : 8.0 G
Repo-baseurl       : file:///misc/cd/AppStream, https://mirrors.aliyun.com/centos/8/AppStream/x86_64/os/, https://mirrors.cloud.tencent.com/centos/8/AppStream/x86_64/os/,
                   : https://mirrors.tuna.tsinghua.edu.cn/cc/8/AppStream/x86_64/os/
Repo-expire        : 172,800 second(s) (last: Tue 07 Feb 2023 10:48:27 PM CST)
Repo-filename      : /etc/yum.repos.d/base.repo

Repo-id            : xyyBaseOS
Repo-name          : BaseOS
Repo-revision      : 8.5.2111
Repo-distro-tags      : [cpe:/o:centos:centos:8]:  , 8, C, O, S, e, n, t
Repo-updated       : Sat 13 Nov 2021 09:02:30 AM CST
Repo-pkgs          : 1,709
Repo-available-pkgs: 1,707
Repo-size          : 1.2 G
Repo-baseurl       : file:///misc/cd/BaseOS, https://mirrors.aliyun.com/centos/8/BaseOS/x86_64/os/, https://mirrors.cloud.tencent.com/centos/8/BaseOS/x86_64/os/,
                   : https://mirrors.tuna.tsinghua.edu.cn/cc/8/BaseOS/x86_64/os/
Repo-expire        : 172,800 second(s) (last: Tue 07 Feb 2023 10:48:27 PM CST)
Repo-filename      : /etc/yum.repos.d/base.repo

Repo-id            : xyyepel
Repo-name          : epel
Repo-revision      : 1675735701
Repo-updated       : Tue 07 Feb 2023 10:27:36 AM CST
Repo-pkgs          : 9,545
Repo-available-pkgs: 9,545
Repo-size          : 15 G
Repo-baseurl       : https://mirrors.aliyun.com/epel/8/Everything/x86_64/, https://mirrors.cloud.tencent.com/epel/8/Everything/x86_64/, https://mirrors.tuna.tsinghua.edu.cn/epel/8/Everything/x86_64/
Repo-expire        : 172,800 second(s) (last: Tue 07 Feb 2023 10:48:29 PM CST)
Repo-filename      : /etc/yum.repos.d/base.repo

Repo-id            : xyyextras
Repo-name          : extras
Repo-revision      : 1639140985
Repo-updated       : Fri 10 Dec 2021 08:56:25 PM CST
Repo-pkgs          : 38
Repo-available-pkgs: 38
Repo-size          : 426 k
Repo-baseurl       : https://mirrors.aliyun.com/centos/8/extras/x86_64/os, https://mirrors.cloud.tencent.com/centos/8/extras/x86_64/os/, https://mirrors.tuna.tsinghua.edu.cn/centos/8/extras/x86_64/os/
Repo-expire        : 172,800 second(s) (last: Tue 07 Feb 2023 10:48:28 PM CST)
Repo-filename      : /etc/yum.repos.d/base.repo
Total packages: 17,457
[root@centos8 yum.repos.d]#

```


































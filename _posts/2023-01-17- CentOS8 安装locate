


> locate 是一个命令行工具，用来在 Linux 系统中查找文件。它使用一个数据库来存储文件路径信息，这样你就可以使用 locate 命令来快速查找文件而不需要在整个文件系统中进行搜索。



## 1.有关 locate 命令的错误提示

但是当我们使用这个命令的时候有一些错误提示，如下：

```bash
[root@centos8 ~]#local
local      locale     localectl  localedef
#通过TAB键补全，可以发现是没有locate命令的
[root@centos8 ~]#locate
-bash: locate: command not found
```

通过 **updatedb** 来更新数据库的时候也报错，如下：

```shell
[root@centos8 ~]#updatedb
-bash: updatedb: command not found
```



## 2.安装 mlocate 工具

**mlocate** 是一个用来维护这个数据库的工具。它会扫描文件系统并建立一个数据库，里面包含了文件的路径信息。然后你就可以使用 locate 命令来查找文件了。

```shell
[root@centos8 ~]#yum -y install mlocate
Last metadata expiration check: 0:17:25 ago on Tue 17 Jan 2023 11:17:52 AM CST.
Dependencies resolved.
==========================================================================================================================================
 Package                         Architecture                   Version                              Repository                      Size
==========================================================================================================================================
Installing:
 mlocate                         x86_64                         0.26-20.el8                          baseos                         121 k

Transaction Summary
==========================================================================================================================================
Install  1 Package

Total download size: 121 k
Installed size: 393 k
Downloading Packages:
mlocate-0.26-20.el8.x86_64.rpm                                                                            130 kB/s | 121 kB     00:00
------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                     130 kB/s | 121 kB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                  1/1
  Running scriptlet: mlocate-0.26-20.el8.x86_64                                                                                       1/1
  Installing       : mlocate-0.26-20.el8.x86_64                                                                                       1/1
  Running scriptlet: mlocate-0.26-20.el8.x86_64                                                                                       1/1
  Verifying        : mlocate-0.26-20.el8.x86_64                                                                                       1/1

Installed:
  mlocate-0.26-20.el8.x86_64

Complete!

```

当我们安装这个工具以后，使用 **locate** 命令来搜索的话，还是会提示错误，如下：

```shell
[root@centos8 ~]#loca
local      locale     localectl  localedef  locate
#当我们使用tab键补全的时候发现已经有了 locate 这个命令，但是我们使用的时候还是会报错，如下：
[root@centos8 ~]#locate passwd #这条命令会在 mlocate 数据库中查找所有文件名包含 "passwd" 的文件。
locate: can not stat () `/var/lib/mlocate/mlocate.db': No such file or directory
#这说明 mlocate 数据库文件 /var/lib/mlocate/mlocate.db 不存在。
```

可以看到 **locate** 这个命令已经有了，但是使用为什么还会报错呢？

这是因为我们没有使用 **updatedb** 来更新数据库。



## 3.使用updatedb更新数据库

```shell
[root@centos8 ~]#update
update-alternatives     update-crypto-policies  update-mime-database
update-ca-trust         updatedb                update-pciids
#通过tab键发现已经有updatedb这个命令
[root@centos8 ~]#updatedb
[root@centos8 ~]#
```

再次搜索 **passwd**

```shell
[root@centos8 ~]#locate passwd
/data/user_passwd.log
/etc/passwd
/etc/passwd-
/etc/pam.d/passwd
/etc/security/opasswd
/usr/bin/gpasswd
/usr/bin/grub2-mkpasswd-pbkdf2
/usr/bin/htpasswd
/usr/bin/passwd
/usr/lib/firewalld/services/kpasswd.xml
/usr/lib/security/pam_unix_passwd.so
/usr/lib64/security/pam_unix_passwd.so
/usr/sbin/chgpasswd
/usr/sbin/chpasswd
/usr/sbin/lpasswd
/usr/sbin/saslpasswd2
/usr/share/awk/passwd.awk
/usr/share/bash-completion/completions/chpasswd
/usr/share/bash-completion/completions/gpasswd
/usr/share/bash-completion/completions/htpasswd
/usr/share/bash-completion/completions/ldappasswd
/usr/share/bash-completion/completions/passwd
/usr/share/bash-completion/completions/smbpasswd
/usr/share/doc/passwd
/usr/share/doc/passwd/AUTHORS
/usr/share/doc/passwd/ChangeLog
/usr/share/doc/passwd/NEWS
/usr/share/licenses/passwd
/usr/share/licenses/passwd/COPYING
/usr/share/locale/ar/LC_MESSAGES/passwd.mo
/usr/share/locale/as/LC_MESSAGES/passwd.mo
/usr/share/locale/ast/LC_MESSAGES/passwd.mo
/usr/share/locale/bg/LC_MESSAGES/passwd.mo
/usr/share/locale/bn/LC_MESSAGES/passwd.mo
/usr/share/locale/bn_IN/LC_MESSAGES/passwd.mo
/usr/share/locale/bs/LC_MESSAGES/passwd.mo
/usr/share/locale/ca/LC_MESSAGES/passwd.mo
/usr/share/locale/cs/LC_MESSAGES/passwd.mo
/usr/share/locale/cy/LC_MESSAGES/passwd.mo
/usr/share/locale/da/LC_MESSAGES/passwd.mo
/usr/share/locale/de/LC_MESSAGES/passwd.mo
/usr/share/locale/el/LC_MESSAGES/passwd.mo
/usr/share/locale/en_GB/LC_MESSAGES/passwd.mo
/usr/share/locale/es/LC_MESSAGES/passwd.mo
/usr/share/locale/et/LC_MESSAGES/passwd.mo
/usr/share/locale/eu/LC_MESSAGES/passwd.mo
/usr/share/locale/fa/LC_MESSAGES/passwd.mo
/usr/share/locale/fi/LC_MESSAGES/passwd.mo
/usr/share/locale/fr/LC_MESSAGES/passwd.mo
/usr/share/locale/gl/LC_MESSAGES/passwd.mo
/usr/share/locale/gu/LC_MESSAGES/passwd.mo
/usr/share/locale/he/LC_MESSAGES/passwd.mo
/usr/share/locale/hi/LC_MESSAGES/passwd.mo
/usr/share/locale/hr/LC_MESSAGES/passwd.mo
/usr/share/locale/hu/LC_MESSAGES/passwd.mo
/usr/share/locale/hy/LC_MESSAGES/passwd.mo
/usr/share/locale/id/LC_MESSAGES/passwd.mo
/usr/share/locale/is/LC_MESSAGES/passwd.mo
/usr/share/locale/it/LC_MESSAGES/passwd.mo
/usr/share/locale/ja/LC_MESSAGES/passwd.mo
/usr/share/locale/ka/LC_MESSAGES/passwd.mo
/usr/share/locale/kn/LC_MESSAGES/passwd.mo
/usr/share/locale/ko/LC_MESSAGES/passwd.mo
/usr/share/locale/ku/LC_MESSAGES/passwd.mo
/usr/share/locale/lo/LC_MESSAGES/passwd.mo
/usr/share/locale/mk/LC_MESSAGES/passwd.mo
/usr/share/locale/ml/LC_MESSAGES/passwd.mo
/usr/share/locale/mr/LC_MESSAGES/passwd.mo
/usr/share/locale/ms/LC_MESSAGES/passwd.mo
/usr/share/locale/my/LC_MESSAGES/passwd.mo
/usr/share/locale/nb/LC_MESSAGES/passwd.mo
/usr/share/locale/nds/LC_MESSAGES/passwd.mo
/usr/share/locale/nl/LC_MESSAGES/passwd.mo
/usr/share/locale/nn/LC_MESSAGES/passwd.mo
/usr/share/locale/or/LC_MESSAGES/passwd.mo
/usr/share/locale/pa/LC_MESSAGES/passwd.mo
/usr/share/locale/pl/LC_MESSAGES/passwd.mo
/usr/share/locale/pt/LC_MESSAGES/passwd.mo
/usr/share/locale/pt_BR/LC_MESSAGES/passwd.mo
/usr/share/locale/ro/LC_MESSAGES/passwd.mo
/usr/share/locale/ru/LC_MESSAGES/passwd.mo
/usr/share/locale/si/LC_MESSAGES/passwd.mo
/usr/share/locale/sk/LC_MESSAGES/passwd.mo
/usr/share/locale/sl/LC_MESSAGES/passwd.mo
/usr/share/locale/sq/LC_MESSAGES/passwd.mo
/usr/share/locale/sr/LC_MESSAGES/passwd.mo
/usr/share/locale/sr@latin/LC_MESSAGES/passwd.mo
/usr/share/locale/sv/LC_MESSAGES/passwd.mo
/usr/share/locale/ta/LC_MESSAGES/passwd.mo
/usr/share/locale/te/LC_MESSAGES/passwd.mo
/usr/share/locale/tr/LC_MESSAGES/passwd.mo
/usr/share/locale/uk/LC_MESSAGES/passwd.mo
/usr/share/locale/ur/LC_MESSAGES/passwd.mo
/usr/share/locale/vi/LC_MESSAGES/passwd.mo
/usr/share/locale/wa/LC_MESSAGES/passwd.mo
/usr/share/locale/zh_CN/LC_MESSAGES/passwd.mo
/usr/share/locale/zh_TW/LC_MESSAGES/passwd.mo
/usr/share/man/cs/man1/gpasswd.1.gz
/usr/share/man/de/man1/gpasswd.1.gz
/usr/share/man/de/man8/chgpasswd.8.gz
/usr/share/man/de/man8/chpasswd.8.gz
/usr/share/man/fr/man1/gpasswd.1.gz
/usr/share/man/fr/man8/chgpasswd.8.gz
/usr/share/man/fr/man8/chpasswd.8.gz
/usr/share/man/hu/man1/gpasswd.1.gz
/usr/share/man/it/man1/gpasswd.1.gz
/usr/share/man/it/man8/chgpasswd.8.gz
/usr/share/man/it/man8/chpasswd.8.gz
/usr/share/man/ja/man1/gpasswd.1.gz
/usr/share/man/ja/man1/passwd.1.gz
/usr/share/man/ja/man8/chpasswd.8.gz
/usr/share/man/man1/gpasswd.1.gz
/usr/share/man/man1/grub2-mkpasswd-pbkdf2.1.gz
/usr/share/man/man1/htpasswd.1.gz
/usr/share/man/man1/lpasswd.1.gz
/usr/share/man/man1/openssl-passwd.1ssl.gz
/usr/share/man/man1/passwd.1.gz
/usr/share/man/man1/sslpasswd.1ssl.gz
/usr/share/man/man3/passwd2des.3.gz
/usr/share/man/man5/passwd.5.gz
/usr/share/man/man8/chgpasswd.8.gz
/usr/share/man/man8/chpasswd.8.gz
/usr/share/man/pt_BR/man1/gpasswd.1.gz
/usr/share/man/ru/man1/gpasswd.1.gz
/usr/share/man/ru/man8/chgpasswd.8.gz
/usr/share/man/ru/man8/chpasswd.8.gz
/usr/share/man/zh_CN/man1/gpasswd.1.gz
/usr/share/man/zh_CN/man8/chgpasswd.8.gz
/usr/share/man/zh_CN/man8/chpasswd.8.gz
/usr/share/man/zh_TW/man8/chpasswd.8.gz
/usr/share/nmap/scripts/http-passwd.nse
/usr/share/vim/vim80/ftplugin/passwd.vim
/usr/share/vim/vim80/syntax/passwd.vim
/var/lib/sss/mc/passwd
[root@centos8 ~]#
```

以后就可以正常使用locate了！！！




> 举个例子：当输入timedatectl list-timezones这个命令时，timedatectl 是会提示的，但是当我们输入list以及后面的部分再次按下tab键是没有任何提示的，这样在我们运维的时候很不方便。
> 
那么下面就用到一个功能很强大的工具：bash命令补全工具（bash-completion）

```java
#安装Linux命令补全工具
yum -y install bash-completion
#执行bash或者reboot重启系统
bash
```

```bash
#如果上述的命令执行了有问题可以执行功能下面的命令
yum install epel-release -y
yum install bash-completion bash-completion-extras -y 
```
如果没有安装bash-completion这个工具包输入timedatectl并按tab显示是下方样式：
![在这里插入图片描述](https://img-blog.csdnimg.cn/f201e8efbf644054a4529b7ffad693ed.png)
安装之后呢？-当然会有真正的命令提示出来
![在这里插入图片描述](https://img-blog.csdnimg.cn/721ea15f2e14414e8707156dec783fcb.png)
当然输入别的也是可以提示出来的，比如：systemctl
![在这里插入图片描述](https://img-blog.csdnimg.cn/fa09c82160db49778638d5bccd350cb9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAWGluZ1l1eXVfQ29kZXI=,size_20,color_FFFFFF,t_70,g_se,x_16)


当我们安装好这个补全工具的时候，使用tab键就会有命令提示了
![在这里插入图片描述](https://img-blog.csdnimg.cn/ea0bbbcd39e2481a8b9f25a5592ba87e.png)

```c
#Ubuntu也可以进行命令补全
apt install -y bash-completion
```

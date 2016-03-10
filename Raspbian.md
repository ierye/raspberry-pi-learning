# Raspbian

##一、树莓派上手配置

###0.准备
- 树莓派主板、刷好Raspbian系统的tf存储卡  
- 5V 2A microUSB充电器一个  
- 路由器一个、网线一条  
- Mac / Windows / Linux 电脑一台  

###1.树莓派SSH登录
树莓派与电脑在同一网段下，查找树莓派的IP地址（本人的是192.168.1.152），打开Mac或者Linux终端（Windows电脑自行百度SSH登录方法），输入命令`ssh pi@192.168.1.152`(也可以指定端口号`ssh pi@192.168.1.152 -p 22`)，默认密码是：***raspberry***，即可ssh登录成功。

###2.初始化树莓派设置
####2.1 更新软件源与本地软件
1. 更新软件源  
   `sudo apt-get update`  
   通过以上命令升级完成后重启。  
2. 更新本地软件  
   `sudo apt-get upgrade`

####2.2 树莓派配置  
在SSH终端输入`sudo raspi-config`,然后配置以下选项：
1. expand_rootfs - 将根分区扩展到整张SD卡;
2. change_pass - 更改登录密码，默认的用户名是pi，密码是raspberry;
3. change_timezone - 更改时区, 选择Asia – Shanghai;
4. configure_keyboard - 选English（US）;
5. change_local - 更改语言设置，选择`en_US.UTF-8`和`zh_CN.UTF-8`
设置完成后，选择**Finish**，会提示是否重启，选择**Yes**.

####2.3 树莓派开启SSH root登录权限  
1. Raspberry Pi 的 Raspbian 系统  
   默认用户是：***pi***  
   默认密码是：***raspberry***  

2. 如果需要开启root登录权限，可登录***pi*** 用户后，在命令行下执行  
   `sudo passwd root`  
   根据提示输入两遍密码，然后再执行以下命令解锁root账户  
   `sudo passwd --unlock root`  
   修改`/etc/ssh/sshd_config`文件，将`PermitRootLogin without-password`改为`PermitRootLogin yes`，然后执行`sudo reboot`重启，即可使用root账户登录。
3. 关闭root用户ssh登录  
   树莓派默认安装了SSH服务，SSH登录的方式为：`ssh pi@树莓派ip地址`，默认密码为：`raspberry`
   为了安全, 推荐关闭root用户ssh登录, 方法为修改`/etc/ssh/sshd_config`, 将 `PermitRootLogin yes`改为`PermitRootLogin no`
   终端重启ssh服务: `service ssh restart`

####2.4 树莓派在Shell中切换用户  
1. Shell介绍  
大部分Linux发行版的默认账户通常是普通用户，而更改系统文件或者执行某些命令，需要root身份才能进行，这就需要从当前用户切换到root用户,Linux中切换用户的命令是`su`或`su -`.  
<br />
2. `su`命令和`su -`命令的主要区别如下：
  - `su`只是切换root身份，但Shell环境仍然是普通用户的Shell，执行`pwd`命令，会发现工作目录仍然是普通用户的工作目录.
  - `su -`可以连**用户**和**Shell环境**一起切换成root身份，只有切换了Shell环境才不会出现PATH环境变量错误，执行`su -`命令切换为root身份以后，执行`pwd`命令，会发现工作目录变成了root的工作目.  
<br />
3. 切换用户命令  
  1. 从普通用户***pi***切换到***root***用户  
     执行`su`或者`su -`或者`su - root`命令
  2. 从***root***用户切换到普通用户***pi***  
     执行`su - pi`命令

####2.5 树莓派安装VNC  
1. 安装VNC
   ```bash
   pi@raspberrypi:~ $ sudo apt-get install tightvncserver
   ```
2. 设置VNC密码。输入以下命令设置密码后，会询问是否设置一个查看(view-only)密码，输入**NO**即可。  
   ```bash
   pi@raspberrypi:~ $ vncpasswd
   ```
3. 设置开机启动。在/etc/init.d/中创建一个文件，如**tightvncserver**。
   ```bash
   pi@raspberrypi:~ $ sudo nano /etc/init.d/tightvncserver
   ```
   输入如下内容(以下代码第一行必不可少)：
   ```sh
   #!/bin/sh
   ### BEGIN INIT INFO
   # Provides:          tightvncserver
   # Required-Start:    $local_fs
   # Required-Stop:     $local_fs
   # Default-Start:     2 3 4 5
   # Default-Stop:      0 1 6
   # Short-Description: Start/stop tightvncserver
   ### END INIT INFO
   
   # More details see:
   # http://www.penguintutor.com/linux/tightvnc
   
   ### Customize this entry
   # Set the USER variable to the name of the user to start tightvncserver under
   export USER='pi'
   ### End customization required
   
   eval cd ~$USER
   
   case "$1" in
     start)
       # 启动命令行。此处自定义分辨率、控制台号码或其它参数。
       su $USER -c '/usr/bin/tightvncserver -depth 16 -geometry 1280x800 :1'
       echo "Starting TightVNC server for $USER "
       ;;
     stop)
       # 终止命令行。此处控制台号码与启动一致。
       su $USER -c '/usr/bin/tightvncserver -kill :1'
       echo "Tightvncserver stopped"
       ;;
     *)
       echo "Usage: /etc/init.d/tightvncserver {start|stop}"
       exit 1
       ;;
   esac
   exit 0
   ```
   按ctrl+o保存后按回车，然后ctrl+x退出nano编辑器。  
4. 给tightvncserver文件加执行权限，并更新开机启动列表
   ```bash
   pi@raspberrypi:~ $ sudo chmod 755 /etc/init.d/tightvncserver
   pi@raspberrypi:~ $ sudo update-rc.d tightvncserver defaults
   ```
5. 关闭并重启vncserver
   ```bash
   pi@raspberrypi:~ $ sudo service tightvncserver stop
   pi@raspberrypi:~ $ sudo service tightvncserver start
   ```
6. 终止指定编号的VNC控制台
   ```bash
   pi@raspberrypi:~ $ sudo tightvncserver -kill :1
   ```
7. 查看正在运行的控制台列表：
   ```bash
   pi@raspberrypi:~ $ sudo ps ax | grep Xtightvnc | grep -v grep
   ```

###2.6 树莓派安装SVN



###2.7 树莓派安装GIT
git帮助文档http://git-scm.com/book/zh


###2.8 上传文件到树莓派
`$ scp -P 36397 ~/.ssh/id_rsa.pub root@2-ye.wicp.net:/root/.ssh/authorized_keys2`  
`$ scp -o port=36397 ~/.ssh/id_rsa.pub root@2-ye.wicp.net:/root/.ssh/authorized_keys2`  
`$ scp -P 36397 -r ~/.* root@2-ye.wicp.net:/root/.ssh/authorized_keys2`  
命令解释如下:  
1.`scp`上传写入命令（***scp命令可以将文件上传到服务器指定目录下，也可将文件写入到指定的服务器文件中***）  
2.`-P`指定端口号，P大写  
3.`36396`服务器端口号  
4.`~/.ssh/id_rsa.pub`要上传的本机文件的路径  
5.`root@2-ye.wicp.net:/root/.ssh/authorized_keys`***root***服务器用户名，***2-ye.wicp.net***服务器地址，***/root/.ssh/authorized_keys***将要上传文件在服务器的存储路径或者目标文件，注意不要丢了`@`符和`:`符。  
6.`-r`将目录下的目录递归拷贝  
7.`.*`将隐藏文件也拷贝过去  

###2.9 树莓派SSH公钥私钥生成
`ssh`无密码登录要使用公钥与私钥。linux下可以使用`ssh-keygen`生成公钥/私钥对。
示例:
有电脑A(192.168.1.100)，电脑B(192.168.1.200)。
现想电脑A通过ssh免密码登录到电脑B。
首先以root账户登陆为例。  
1.在A机下生成公钥/私钥对。  
```
[root@A ~]# ssh-keygen -t rsa -P ''
```  
命令解释:`-P`表示密码，`-P ''` 就表示空密码，也可以不用`-P`参数，直接执行命令`ssh-keygen -t rsa`，这样就要连续三次回车，用`-P`只需一次回车。  
该命令将在`~/.ssh`目录下面产生一对密钥***id_rsa***和***id_rsa.pub***。  
一般采用的ssh的rsa密钥:  
***id_rsa***私钥  
--该用户默认的DSA身份认证私钥(SSH-2)  
--此文件的权限应当至少限制为"600"  
--生成密钥的时候可以指定采用密语来加密该私钥(3DES)  
--ssh将在登录的时候读取这个文件。  
***id_rsa.pub***公钥  
--该用户默认的DSA身份认证公钥(SSH-2)。此文件无需保密。  
--此文件的内容应该添加到所有DSA目标主机的 `~/.ssh/authorized_keys`文件中。  
下述命令产生不同类型的密钥  
`ssh-keygen -t dsa`  
`ssh-keygen -t rsa`  
`ssh-keygen -t rsa1`  
2.把电脑A下的`~/.ssh/id_rsa.pub`复制到电脑B的 `/root/.ssh/authorized_keys`文件里（注意authorized_keys是文件不是文件夹）。如果B上没有***.ssh***文件夹则手动创建，创建好 /root/.ssh 这个目录，用`scp`命令将电脑A的***id_rsa.pub***文件中的内容复制到电脑B的***authorized_keys***文件中。  
```
[root@A ~]# scp ~/.ssh/id_rsa.pub root@192.168.1.200:/root/.ssh/authorized_keys
```
回车执行以上命令后输入电脑B的密码然后回车复制完成。   
3.尝试ssh登录如果依然需要输入密码,请注意以下问题。  
***.ssh***目录的权限要必须为700  
***authorized_keys***文件的权限要必须为600   
```
[root@B ~]# chmod 700 /root/.ssh
[root@B ~]# chmod 600 /root/.ssh/authorized_keys
```
注:**authorized_keys**和**authorized_keys2**的区别:  
SSH protocols 1.3和1.5 使用`$HOME/.ssh/authorized_keys`文件   
SSH protocol 2.0 使用`$HOME/.ssh/authorized_keys2`文件  
在***OpenSSH version 3***以后，已不赞成使用***authorized_keys2***（虽然仍能用）。  
所有的公钥都应该放在***authorized_keys***文件中。

总结:
ssh登录的电脑(A)需要有私钥，被登录的电脑(B)要有登录电脑(A)的公钥。这个公钥/私钥对一般在私钥宿主机(电脑A)产生。上面是用rsa算法的公钥/私钥对，当然也可以用dsa(对应的文件是***id_dsa***，***id_dsa.pub***)。
如果想让A、B两台电脑无密码相互登录，那么电脑B以上面同样的方式配置即可。

###3.0 SSH-KeyGen的用法
 
假设电脑A为客户机器，电脑B为目标机；

要达到的目的：
电脑A通过ssh登录电脑B无需输入密码；
加密方式选rsa|dsa均可，默认dsa.

做法：  
1.登录A机器  
2.`ssh-keygen -t [rsa|dsa]`，将会生成密钥文件和私钥文件 ***id_rsa***,***id_rsa.pub***或***id_dsa***,***id_dsa.pub***  
3.将***.pub***文件复制到B机器的***.ssh***目录，登录电脑B并执行命令`cat id_dsa.pub >> ~/.ssh/authorized_keys`  
4、大功告成，从电脑A登录电脑B的目标账户就不再需要密码了。
 
 
 
ssh-keygen做密码验证可以使在向对方机器上ssh ,scp不用使用密码.
具体方法如下:
ssh-keygen -t rsa
然后全部回车,采用默认值.

这样生成了一对密钥，存放在用户目录的~/.ssh下。
将公钥考到对方机器的用户目录下，并拷到~/.ssh/authorized_keys中。

要保证.ssh和authorized_keys都只有用户自己有写权限。否则验证无效。












<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />

####科普知识 Linux 环境变量  
1. Linux 环境变量介绍：  
环境变量实际上就是用户运行环境的参数集合，linux是一个多用户操作系统，而且每个用户登陆后，都会有一个专有的运行环境。通常每个用户默认的环境都是相同的，而这个默认环境实际上都是一组环境变量的定义，用户可以对自己的运行环境进行定制，其方法就是修改相应的系统环境变量。  
<br />
2. 常见的环境变量
  - PATH 系统路径
  - HOME 系统根目录
  - HISTSIZE 保存历史命令记录的条数
  - LOGNAME 当前用户的登陆名
  - HOSTNAME 主机的名称，若应用程序要用到主机名的话，通常是从这个变量中获取的
  - SHELL 当前用户用的Shell类型
  - LANG/LANGUGE 语言相关的环境变量，使用多种语言的用户可以修改此环境变量
  - MAIL 当前用户的邮件存放目录  
<br />
3. 查看和设置环境变量的方法：
  1. 查看环境变量
    - 通过`echo`命令显示指定的环境变量。如`echo $HOME`.
    - 通过`env`命令显示所有的或指定的环境变量。如`env`、`env|grep HOME`.
    - 通过`export`命令显示所有的或指定的环境变量。如`export`、`export|grep HOME`.
    - 通过`set`命令显示所有本地定义的Shell变量或指定的变量。如`set`、`set|grep HOME`.  
<br />
  2. 设置环境变量
    - 通过`export`命令设置环境变量。如`export HELLO="HELLO!"`
    - 通过`set`命令设置环境变量。
    - 通过`uset`命令清除环境变量。  
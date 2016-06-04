#COBBLER实验
##实验目的
- 该实验主要要求操作者熟练掌握cobbler的部署，并且通过实验联系理论，了解cobbler的一些工作原理。
- 交作业
##实验要求
1. 在服务器上部署cobbler，可以通过web界面进行访问。
2. 部署完成后可以选择待安装的系统，分别是centos6系统与centos7系统。
3. 部署完以后形成报告。
4. 扩展项：形成cobbler安装的shell脚本，扩展yum仓库部分。

##实验准备
1. 服务器一台，1核CPU，20G硬盘，2G内存，centos7系统，内核3.10.0。
2. 《装机部署规范》
3. centos6和centos7的镜像文件

##实验步骤
###基础实验
- 打开虚拟机，同步一下epel源。
<pre>
rpm -ivh http://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm
</pre>
- 安装系统所要求的必备的软件。
<pre>
yum install -y httpd dhcp cobbler cobbler-web pykickstart xinetd
</pre>
- 启动cobblerd和httpd两个服务
<pre>
systemctl start httpd
systemctl start cobblerd
</pre>
- 检查运行cobbler需要执行修改的项目并且逐步改正。
<pre>cobbler check</pre>
显示8处需要修改。

1. cobbler配置文件的server段要进行更改，从127.0.0.1改为dhcp服务端的ip。
2. cobbler配置文件的next_server段需要进行更改，从127.0.0.1改为tftp服务端的Ip。
3. 在xinetd.d配置文件里面设置将tftp服务器启动。
4. 通过cobbler get-loaders命令来获取一些引导程序。
5. 开启rsync
6. debmirror未开启（centos7系统可以不理）
7. 默认密码太简单
8. 没有找到电源控制工具。

执行下面的命令进行修复
<pre>
sed -i '384 s#server: 127.0.0.1#server: 192.168.56.11#' /etc/cobbler/settings
sed -i '272 s#next_server: 127.0.0.1#next_server: 192.168.56.11#' /etc/cobbler/settings
sed -i '14 s#        disable                 = no#        disable                 = yes#' /etc/xinetd.d/tftp
cobbler get-loaders
systemctl start rsyncd
</pre>
设置新装系统的默认root密码cobler。下面的命令来源于提示7。random-phrase-here为干扰码，可以自行设定。
<pre>
获取密码：openssl passwd -1 -salt 'cobler' 'cobler'
修改配置文件/etc/cobbler/settings的default_password_crypted项
</pre>


同时为了能保证控制cobbler进行配置以助于cobbler能生成正确的dhcp配置文件并启动dhcp
<pre>
使cobbler可以控制dhcp
sed -i '242 s#manage_dhcp: 0#manage_dhcp: 1#' /etc/cobbler/settings
修改默认配置文件
vim /etc/cobbler/dhcp.template 
修改
subnet 192.168.56.0 netmask 255.255.255.0 
option routers             192.168.56.2;
option domain-name-servers 192.168.56.2;
range dynamic-bootp        192.168.56.100 192.168.56.254;
分别为本机Ip的网络地址、掩码、网关、分配的ip段。
</pre>
重启cobbler，并且重新生成配置文件等等。
systemctl restart cobblerd
cobbler sync

- 挂载光盘并且生成profile

<pre>
mount /dev/cdrom /mnt
cobbler import --path=/mnt/ --name=CentOS-7-x86_64 --arch=x86_64
umount /mnt
将光驱内光盘换成6的镜像文件
mount /dev/cdrom /mnt
cobbler import --path=/mnt/ --name=CentOS-6-x86_64 --arch=x86_64
</pre>

- 配置kictstart文件

<pre>
上传文件到
/var/lib/cobbler/kickstarts/
指定kickstart配置文件
cobbler profile edit --name=CentOS-7-x86_64 --kickstart=/var/lib/cobbler/kickstarts/CentOS-7-x86_64.cfg
cobbler profile edit --name=CentOS-6-x86_64 --kickstart=/var/lib/cobbler/kickstarts/CentOS-6-x86_64.cfg
</pre>

- 更改centos7系统装机完毕后的网卡名字
<pre>
cobbler profile edit --name=CentOS-7-x86_64 --kopts='net.ifnames=0 biosdevname=0'
</pre>

- 同步cobbler配置文件
执行命令
<pre>
cobbler sync
</pre>
启动xinetd
<pre>
systemctl start xinetd
</pre>

### 扩展部分

进入cobbler的web界面：同步cobbler配置文件以后，直接在浏览器端输入ip地址后面接/cobbler即可。

- 自定义yum源

寻找yum源 可以直接去mirror.aliyun.com去寻找。
找到了源头以后添加到cobbler里面，以openstack为例:
<pre>
添加一个报告（repo）

cobbler repo add --name=openstack-mitaka --mirror=http://mirrors.aliyun.com/centos/7.2.1511/cloud/x86_64/openstack-mitaka/ --arch=x86_64 --breed=yum
同步
cobbler reposync
</pre>
使新装的系统自动将openstack-mitaka加进去
<pre>
添加repo到相应的profile
 cobbler profile edit --name=Centos-7-x86_64  --repos=http://mirrors.aliyun.com/centos/7.2.1511/cloud/x86_64/openstack-mitaka/ 
修改ks文件使之与客户端同步
%post
systemctl disable postfix.service

$yum_config_stanza
%end
</pre>
- 实现设置好参数实现自动安装

查阅 《装机部署规范》
确定如下参数

1. ip地址 
2. 网关 
3. 掩码 
4. 主机名 
5. 所要安装的系统
6. mac地址（供应商提供）
7. dns

确定参数：

1. Ip地址：192.168.56.12
2. 所要安装的系统：CentOS-7-x86_64
3. hostname：linux-node2.oldboyedu.com
4. mac地址：00:50:56:31:6C:DF
5. 网关：192.168.56.2
6. 掩码：255.255.255.0
7. dns：192.168.56.2

使用命令定义
<pre>
cobbler system add --name=linux-node2.oldboyedu.com --mac=00:50:56:31:6C:DF --profile=CentOS-7-x86_64 \
--ip-address=192.168.56.12 --subnet=255.255.255.0 --gateway=192.168.56.2 --interface=eth0 \
--static=1 --hostname=linux-node2.oldboyedu.com --name-server="192.168.56.2" \
--kickstart=/var/lib/cobbler/kickstarts/CentOS-7-x86_64.cfg
</pre>
##实验总结
1. 可以了解一下api接口的信息
2. 形成脚本化

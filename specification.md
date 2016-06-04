##操作系统安装及初始化规范
> 本规范为初定版，如在工作的使用中出现明显漏洞，请联系1079195011@qq.com



###操作系统安装规范
1. 服务器采购：
服务器采购要在正规的厂商的代理商处采购。

2. 服务器验收并设置raid：服务商提供验收单，运维验收负责人签字，同时让厂家派来的人员做raid

4. 服务器上架：服务器上架。

5. 资产录入：资产录入到cmdb中
 
6. 开始自动化安装：根据cmdb中的信息进行个性化安装


##系统初始化规范
###初始化操作
* 设置DNS 192.168.56.111和192.168.56.112
* 安装zabbix agent：server 192.168.56.11
* 安装saltstace minion:saltstack master：192.168.56.11
* history记录时间 
<pre>
export PROMPT_COMMAND='{ msg=$(history 1 | { read x y; echo $y; });logger "[euid=$(whoami)]":$(who am i):[`pwd`]"$msg"; }'
export HISTTIMEFORMAT=%F %T root
</pre>

* 日志记录操作
<pre>
export PROMPT_COMMAND='{ msg=$(history 1 | { read x y; echo $y; });logger "[euid=$(whoami)]":$(who am i):[`pwd`]"$msg"; }'
</pre>
* 内核参数优化
* yum仓库
* 基本优化

 
###目录规范
* 脚本放置目录：/opt/shell
* 脚本日志目录：/opt/shell/log
* 脚本锁文件目录：/opt/shell/lock


###程序服务安装规范
1. 源码安装路径 /usr/local/appname.version
2. 创建软连接 ln -s /usr/local/appname.version /usr/loca/appname
3. 日志文件存放：/var/logs
4. 创建程序同名user作为程序的用户。

###主机名命名规范
**机房名称-项目-角色-服务-集群-节点.域名**
如果某项是完全一致可以省略（比如在单机房的时候 机房名称项就可省略。）

eg：idc01-xxshop-nginx-bj-node1.shop.com

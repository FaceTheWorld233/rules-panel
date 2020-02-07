![](https://raw.githubusercontent.com/Git-Lofter/rules-panel/master/img/01.png)

![](https://raw.githubusercontent.com/Git-Lofter/rules-panel/master/img/02.png)

![](https://raw.githubusercontent.com/Git-Lofter/rules-panel/master/img/03.png)

# 控制端部署：

1. 上传源代码 设置public文件夹为运行目录(注意，文件不包含Master文件夹)。
2. Nginx伪静态配置：

`location /{     if (!-e $request_filename) {       rewrite ^/(.*)$ /index.php/$1 last;       break;     }    }  `

Apache伪静态（.htaccess在public文件夹下)配置：

`<IfModule mod_rewrite.c>`

`RewriteEngine on`

`RewriteCond %{REQUEST_FILENAME} !-d`

`RewriteCond %{REQUEST_FILENAME} !-f`

`RewriteRule ^(.*)$ index.php/$1 [QSA,PT,L]`

`</IfModule>`

3. 设置定时任务

#安装curl

`apt-get install curl -y`

#设定定时任务（5分钟执行一次）

`crontab -e`

#添加定时任务，请替换网址为你自己的

`*/5 * * * * curl https://网址/cron`

4. 导入数据库（test.sql）数据库配置文件app/config.php  默认用户名密码为：admin 123456

# 被控端部署-Golang版(推荐):

代码和执行逻辑重构，大幅降低CPU占用情况，但是不开放源代码，仅提供编译好的二进制程序

本程序会接管iptables的NAT规则，如果您不熟练iptables，请不要使用其他基于iptables的脚本！！！


#开启转发(此处为逗比脚本)：

`wget http://ftp.taoluyun.cc/iptables-pf.sh && chmod +x iptables-pf.sh`

然后执行 `./iptables-pf.sh` 执行选项1安装iptables

#清空本地iptables规则

`iptables -F`

`iptables -t nat -F`

#保存防火墙

CENTOS:

`service iptables save`

Debian:

`iptables-save > /etc/iptables.up.rules`


#上传被控文件至服务器的 `/root`目录下，文件地址：`http://ftp.taoluyun.cc/ip_table`

#加权限

`chmod +x ip_table`

#设定定时任务：

`crontab -e`

`*/5 * * * * /root/ip_table key123 10.0.0.4 https://baidu.com/api`

#参数说明: 

*key123 为 主控面板添加服务器后，分配的key

*10.0.0.4 为主网卡上的IP，查看方法：`ip addr`。如果您的IP为公网IP，并且是动态IP，当IP变动时需要修改此处

*https://baidu.com/api 为您的主控URI，请自行替换为您的域名

*更换Golang需要关闭nodejs被控，以免引起混乱

`pm2 delete 0`

`pm2 save`

然后删除nodejs相关文件即可

# 被控端部署教程-NodeJS版(此版本不推荐使用，建议使用Golang版):

#安装nodejs最新版（centos）

`yum install epel-* -y`

`yum install nodejs -y`

`npm install n -g`

`n latest`

`npm install pm2 -g`

#安装nodejs最新版（debian）

`curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash - && apt-get install nodejs`

`npm install pm2 -g`

#nodejs和pm2安装结束

#pm2开机自启

`pm2 startup`

#新建文件夹iptables_forward

`mkdir iptables_forward`

#将 app.js、package.json两个文件放进去

#修改app.js文件，如下三处，保存

​	`const master_url = "https://baidu.com"  #填写Master URL`

​	`const slave_key = '123456'; #填写节点key`

​	`const nic_ip = '1.1.1.1'; #主网卡上的IP（如果主网卡IP=公网IP时，当IP变动，需更新此处IP，并且重启本进程！！！）`



*Mater URL是主控的网址

*key是主控添加服务器后生成的

*主网卡IP查看方法：`ip addr`

#然后在该文件夹下执行

`npm install` 

#安装iptables转发（逗比转发脚本）

`wget http://ftp.inwang.net/iptables-pf.sh && chmod +x iptables-pf.sh`

#执行iptables转发脚本，执行第一个选项安装iptables

#启动

`pm2 start app.js`

#开机自启

`pm2 save`

#重启服务器 

`reboot`     

#********到此安装结束，可以愉快地使用了

##### #其他命令

`pm2 list`       #查询

`pm2 logs 0`  #查询日志

`pm2 stop 0` #暂停

`pm2 flush`   #清除日志


#### #错误分析

1. 提示文件写入失败：`chown www 网站目录 -R`
2. 添加完服务器却找不到：给用户分配权限
3. 如果你的小鸡是NAT，主网卡ip应该为内网ip(通常为10.开头)
4. 端口不通：放行iptables防火墙。如果是centos 需要卸载firwall启用iptables
5. 如果你的小鸡是NAT，app.js里的ip应该为内网ip

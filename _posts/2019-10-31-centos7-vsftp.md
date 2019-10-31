# CentOS7配置虚拟用户访问并设置SSL加密
环境：服务器IP地址为10.10.10.10  
要求创建用户ftp_user，ftp目录在/var/ftpsite

## 关闭selinux(可选)
vi /etc/selinux/config  
修改enforcing为disabled，保存退出重启服务器

## 安装vsftpd
yum instal vsftpd -y

## 配置虚拟用户目录
mkdir /etc/vsftpd_user_conf   

## 创建虚拟用户
vi /etc/vsftpd_user_conf/vuser  
输入用户名密码，单数行为用户名，双数名为密码  
ftp_user  
123456  
ftp_admin  
123456  
保存退出

## 创建虚拟用户数据库
db_load -T -t hash -f /etc/vsftpd_user_conf/vuser.txt /etc/vsftpd_user_conf/vuser.db  
给数据库文件600权限  
chmod 600 vuser.db

## 配置虚拟用户权限
vi ftp_user  
local_root=/var/ftpsite  
anon_world_readable_only=NO  
anon_upload_enable=YES  
anon_other_write_enable=YES  
anon_mkdir_write_enable=YES  

## 创建系统用户
系统用户用来对应虚拟用户  
useradd vuser -s /sbin/nologin  
设置用户密码  
passwd vuser

## 修改vsftpd的pam文件，使虚拟用户生效
vi /etc/pam.d/vsftpd  
所有内容注释，并添加以下  
auth required    pam_userdb.so  db=/etc/vsftpd_user_conf/vuser  
account required pam_userdb.so  db=/etc/vsftpd_user_conf/vuser  

## 修改vsftpd主配置文件
vi /etc/vsftpd/vsftpd.conf  
修改anonymous_enable为NO  
添加以下  
guest_enable=YES  
guest_username=vuser  
user_config_dir=/etc/vsftpd_user_conf/  
allow_writeable_chroot=YES  

## 启动vsftp服务
systemctl start vsftpd  

## 防火墙放行vsftpd端口
firewall-cmd --add-service=ftp --zone=public --permanent

---
layout:     post   				    # 使用的布局（不需要改）
title:      CentOS7配置Vsftpd虚拟用户访问并配置SSL加密访问
subtitle:   适合用于公网环境的FTP服务器配置
date:       2019-10-31
author:     Yoshiko2 						# 作者
header-img: img/2019-10-31-centos7-vsftp.jpg
catalog: true 						# 是否归档
tags:								#标签
    - Linux
---

环境：服务器IP地址为10.10.10.10
要求创建用户ftp_user，具有读写权限，ftp根目录在/var/ftpsite

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

## 生成证书
cd /etc/pki/tls/certs
make vsftpd.pem
输入信息
cp -r vsftpd.pem /etc/vsftpd/

## 修改vsftpd主配置文件
vi /etc/vsftpd/vsftpd.conf  
修改anonymous_enable为NO  
添加以下  
guest_enable=YES  
guest_username=vuser  
user_config_dir=/etc/vsftpd_user_conf/  
allow_writeable_chroot=YES  

ssl_enable=YES
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
rsa_cert_file=/etc/vsftpd/vsftpd.pem

## 启动vsftp服务
systemctl start vsftpd  

## 防火墙放行vsftpd端口
firewall-cmd --add-service=ftp --zone=public --permanent

# 完成，请用支持SSL加密的ftp客户端连接ftp服务器，记得连接时勾选SSL加密选项

---
title: linux_vsftp
categories: tech
tag: hide
date: 2018-12-19 00:06:13
tags:
---

```
yum install vsftpd db4 db4-utils

vim /etc/vsftpd/vsftpduser.txt

yangwei
goapk2011QWE!@#

chmod 600 /etc/vsftpd/vsftpduser.txt

生成验证文件
db_load  -T -t hash -f /etc/vsftpd/vsftpduser.txt /etc/vsftpd/vsftpduser.db

vim /etc/pam.d/vsftpd
#%PAM-1.0
#session    optional     pam_keyinit.so    force revoke
#auth       required    pam_listfile.so item=user sense=deny file=/etc/vsftpd/ftpusers onerr=succeed
#auth       required    pam_shells.so
#auth       include     password-auth
#account    include     password-auth
#session    required     pam_loginuid.so
#session    include     password-auth
auth required /lib64/security/pam_userdb.so db=/etc/vsftpd/vsftpduser
account required /lib64/security/pam_userdb.so db=/etc/vsftpd/vsftpduser

vim /etc/vsftpd/vsftpd.conf
anonymous_enable=NO
chroot_local_user=YES
xferlog_file=/var/log/vsftpd.log

最后行添加
listen_port=17221
guest_enable=YES
guest_username=daemon
user_config_dir=/etc/vsftpd/vsftpd_user_conf
virtual_use_local_privs=yes
pasv_enable=yes
pasv_min_port=50000
pasv_max_port=60000
max_per_ip=12
listen_address=192.168.1.168

xferlog_file=/var/log/vsftpd.log

touch /var/log/vsftpd.log

mkdir /etc/vsftpd/vsftpd_user_conf
vim  /etc/vsftpd/vsftpd_user_conf/newspartment
write_enable=YES
anonymous_enable=NO
anon_world_readable_only=NO
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
local_umask=022
download_enable=Yes
local_root=/data2/archives/rom

wget ftp://114.66.201.40:17221/total_x15_forum_post.tar.gz --ftp-user=bbsguest --ftp-password=guestzhi2014bbs
wget ftp://bbsguest@114.66.201.40:17221/total_x15_forum_post.tar.gz -ftp-password=guestzhi2014bbs

ftp服务器列表
192.168.1.18(192.168.3.118)
192.168.1.242
192.168.1.32(192.168.3.32)
192.168.1.134
192.168.1.146(192.168.3.156)
192.168.1.22
192.168.1.168

/usr/local/memcached-1.4.5/bin/memcached -m 4096 -c 5000 -u root -p 11211 -d
/usr/local/memcached-1.4.5/bin/memcached -m 4096 -c 5000 -u root -p 12000 -d

awk -F ',' 'BEGIN{sum=0 ;count=0}{if ($(NF-11) == 2 && $NF == 0 && $3 == "1.6.1_1_1") {sum +=$5; count++;} } END {print "sum="sum" count="count " avg="sum/count}'
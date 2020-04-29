---
title: port check
categories: python
tag: hide
date: 2020-03-16 19:29:08
tags:
---

python2 检查服务器端口连通性

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
import socket
import threading
import smtplib
import time
from email.mime.text import MIMEText
from email.header import Header
log = open("/web/monitor/portcheck.log", "a+")
log.writelines(time.strftime("%Y-%m-%d %X",time.localtime()))
log.writelines('\n')

def sendEmail(ip): 
    sender = 'monitor@mail.com'
    receivers = ['recieve@mail.com'] 
     
    message = MIMEText(ip, 'plain', 'utf-8')
    message['From'] = Header("monitor@ag866.com", 'utf-8')   
    message['To'] =  Header("monitor@ag866.com", 'utf-8')   
     
    subject = 'port 80 can not connect'
    message['Subject'] = Header(subject, 'utf-8')
     
     
    try:
        smtpObj = smtplib.SMTP('mail.ag866.com')
        smtpObj.sendmail(sender, receivers, message.as_string())
        #print "send seccess"
	log.writelines("Email send seccess")
    except smtplib.SMTPException:
        #print "send error"
	log.writelines("Email send error")

def check(ip):
	global sendstr
	socket.setdefaulttimeout(2)
	sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	result = sock.connect_ex((ip,80))
	if result == 0:
#		print ip,"Port is open"
		pass
	else:
#		print ip,"Port is not open"
		if sendstr == '':
			sendstr = ip
		else:
			sendstr = sendstr + ip
		log.writelines(ip)
	sock.close()
	return sendstr
sendstr=''
ojb=[]
with open('/web/monitor/ip_list', 'r') as f:
	while True:
		ip = f.readline()
		if not ip:
			break
			pass
		ip = threading.Thread(target=check,args=(ip,)) 
		ip.setDaemon(True)
		ip.start()
		ojb.append(ip)
for o in ojb:
	o.join()
sendEmail(sendstr)
log.close()
```

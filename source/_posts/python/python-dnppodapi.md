---
title: Dnspod api批量操作 
categories: python
tag: hide
date: 2019-9-10 22:06:53
tags:
---

### 功能写的比较多主要功能如下：


	1. 列出此账号域名、域名搜索 (默认选项，回车继续)
	2. 批量查询域名记录（或者包含哪些关键字）
	3. 批量增加域名
	4. 批量增加记录
	5. 批量修改记录（目标域名必须有相同记录）
	6. 批量删除记录
	7. 批量删除域名
	8. 批量获取日志
	9. 获取D监控日志

```
#!/usr/local/python2.7.5/bin/python2.7
# encoding=utf8  
import re
import sys
reload(sys)  
sys.setdefaultencoding('utf8')
import urllib2
import urllib
import json
import time
import socket
import os
sys.tracebacklimit = 0


'''
功能：此脚本主要处理批量操作，以便批量管理域名
使用方法：直接运行改程序，会逐步提示用法
'''

class dnsPodapi:
	def __init__(self):
		self.public_dic={}
		#self.public_dic["login_token"]=("%s,%s" % ('94266','b7f39e603091f2f33e2b998f2c9b79bb'))
		self.public_dic["format"]="json"
		#self.public_dic["record_type"]="CNAME"
		self.headers={}
		self.headers["User-Agent"]="Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.113 Safari/537.36"
		self.domainlisturl = 'https://dnsapi.cn/Domain.List'
		self.searchurl = 'https://dnsapi.cn/Record.List'
		self.muticreate = 'https://dnsapi.cn/Batch.Domain.Create'
		self.mutirecord = 'https://dnsapi.cn/Batch.Record.Create'
		self.mutimodify = 'https://dnsapi.cn/Batch.Record.Modify'
		self.mutidel = 'https://dnsapi.cn/Record.Remove'
		self.mutidel_domain = 'https://dnsapi.cn/Domain.Remove'
		self.getlog = 'https://dnsapi.cn/Domain.Log'
		self.getmonitorlog = 'https://dnsapi.cn/Monitor.Getdowns'
		self.record_dic = {}
		self.re_list = []
		self.choose_account = {}


#执行
	def execApi(self,url,args=''):
		try:
			info=self.public_dic.copy()
			data=urllib.urlencode(info)  + args
			#print data
			request=urllib2.Request(url,headers=self.headers,data=data)
			response=urllib2.urlopen(request,timeout=5)
			dnsJson=json.load(response)
			#print json.dumps(dnsJson,indent=2)
	#		print dnsJson['status']['message']
			return dnsJson
		except  Exception,e:
			print e

#查询域名
	def listDomain(self,auto='False',input_domain=None):
		self.re_list = {}
		domian_sum = 0
		self.public_dic["offset"]='0'
		domain_list = []
		self.re_list = []
		while 1:
			domains = self.execApi(self.domainlisturl)
			#print json.dumps(domains,indent=2)
			if auto == 'False':
				print '\n\n#####################################################################################'
			for d in domains['domains']:
				if auto == 'False':
					print d['name']
				domain_list.append([d['name'],d['id']])
				self.re_list.append([d['name'],d['id']])
				domian_sum = domian_sum + 1

			if len(domains['domains']) >= 3000:
				self.public_dic["offset"] = int(self.public_dic["offset"]) + 3000
			else:
				print "Totle Domains:",domian_sum
				break
		while 1:
			if auto == 'False':
				input_domain = raw_input('搜索域名例如（aaa.com,bbb.com）,回车则选中以上域名,列出该账户全部域名:')
			
			if input_domain:
				self.re_list = []
				input_domain = input_domain.strip()
				if input_domain.find(',') != -1:
					input_domain = input_domain.split(',')
				else:
					input_domain = [input_domain]

				print '\n\n搜索结果：'
				for rec in domain_list:
					for i in input_domain:
						#if re.search(i.decode('utf-8'),rec[0]):
						if rec[0] == i.decode('utf-8'):
							self.re_list.append(rec)
							print rec
				print 'Input domians:',len(input_domain)
				print 'Total domains:',len(self.re_list)
				#print self.re_list
				if auto == 'True':
					break
			elif input_domain == '':
				break
		return self.re_list

#匹配搜索结果
	def match_item(self,domain,rec,input_search):
		record = rec['name']
		domain = tuple(domain)

		'''input_search = name=@|www|m|vipm|vip,
							type=A,
							value=1.1.1.1'''
		for input_item in input_search:
			input_filters = input_item.split('=')
			if input_filters[1].find('|') != -1:
				num = 1
				for input_match in input_filters[1].split('|'):
					#if re.search(rec[input_filters[0]],input_match):
					if rec[input_filters[0]] == input_match:
						break
					if num == len(input_filters[1].split('|')):
						return
					num = num + 1
			else:
				if rec[input_filters[0]] != input_filters[1]:
					return

		#print 'status:', rec['status'], '\t', 'id:', rec['id'], 'type:', rec['type'], 'value:', rec['value'], 'name:', rec['name']
		print 'status:', rec['status'], '\t', 'id:', rec['id'], 'type:', rec['type'], 'value:', rec['value'], '线路：', rec['line'], 'name:', rec['name']
		if domain in self.record_dic:
			if record in self.record_dic[domain]:
				self.record_dic[domain][record].append(rec['id'])
			else:
				self.record_dic[domain].update({record:[rec['id']]})
		else:
			self.record_dic.update({domain:{record:[rec['id']]}})

		return self.record_dic

#查询记录
	def listRecord(self,auto='False',re_search='True',input_search=None):
		self.public_dic["length"]=3000
		if re_search == 'True':
			self.listDomain()
		while 1:
			print '\n\n'
			if auto == 'False':
				input_search = raw_input('''
**多条件查询,支持复合查询，Ex：name=www|@,type=CNAME,value=abc.com.,line=国内,回车列出所有记录,(注意默认条件为：name=@|www|m|vip|vipm)
回复0，重新选择域名
回复y选择以上结果:''')
			if len(input_search) >0 and re.search('name',input_search) or re.search('type',input_search) or re.search('value',input_search) or re.search('line',input_search):
				input_search = input_search.strip()
				try:
					input_search = input_search.split(',')
				except:
					input_search = list(input_search)
			elif input_search  == 'y':
				break
			elif input_search  == '0':
				return self.listRecord()
			else: 
				input_search  = 'all'

			self.record_dic = {}
			for domain in self.re_list:
				try:
					#print domain[0]
					self.public_dic["domain"] = domain[0]
					self.public_dic["offset"]='0'
					res = self.execApi(self.searchurl)
					#print res
					#os.system('clear')
					print '\n', '###################################################################################################', domain[0]
					if res['status']['code'] == '1' and input_search  == 'all':
						for rec in res['records']:
							if rec['type'] != u'NS' and rec['type'] != u'MX':
								#print 'status:', rec['status'], '\t', 'id:', rec['id'], 'type:', rec['type'], 'value:', rec['value'], 'name:', rec['name']
								print 'status:', rec['status'], '\t', 'id:', rec['id'], 'type:', rec['type'], 'value:', rec['value'], '线路：', rec['line'], 'name:', rec['name']
					elif res['status']['code'] == '1':
						# print res
						for rec in res['records']:
							if rec['type'] != u'NS' and rec['type'] != u'MX':
								self.match_item(domain,rec,input_search)
					else:
						print "域名错误：", records['status']
						continue
				except  Exception,e:
					print e
					continue
			if auto == 'True':
				break
		return self.record_dic

#批量增加域名
	def addMutidomain(self):
		input_add_domain = raw_input('输入需要添加的域名，用逗号隔开：')
		print input_add_domain
		raw_input("请确认输入域名，回车继续")
		self.public_dic["domains"]=input_add_domain
		res = self.execApi(self.muticreate)
		print res['status']['message']

#批量增加记录
	def addMutirecord(self):
		add_record_list = []
		relist = []
		tdic = {}
		self.listDomain()
		input_add_type = raw_input('1、CNAME 2、A记录，输入编号(回车默认A记录)')
		if input_add_type and int(input_add_type) in [1,2]:
			input_add_type = int(input_add_type)
			if input_add_type == 1:
				tdic['record_type']='CNAME'
			elif  input_add_type == 2:
				tdic['record_type']='A'
		else:
			tdic['record_type']='A'
		input_add_area = raw_input('1、默认 2、国内 3、海外 (回车即默认) :')
		if input_add_area:
			input_add_area = int(input_add_area)
		else:
			input_add_area = 1

		input_add_record = raw_input('输入需要添加的二级域名，解析地址用等号隔开，每条记录用逗号隔开例如 "www=8.8.8.8,www=123.2.2.2,app=2.2.2.2" ：')
		input_add_record = input_add_record.strip()
		input_add_record = input_add_record.split(',')
		for i in input_add_record:
			record = i.split('=')
			add_record_list.append([record[0],record[1]])
		print "\n\n目标域名如下 \t###############################################################################"
		for domain in self.re_list:
			#print domain[0]
			if 'domain_id' in self.public_dic:
				self.public_dic['domain_id'] = self.public_dic['domain_id'] +','+str(domain[1])
			else:
				self.public_dic['domain_id'] = str(domain[1])


		if input_add_area == 1:
			tdic['record_line']='默认'
		elif input_add_area == 2:
			tdic['record_line']='国内'
		elif input_add_area == 3:
			tdic['record_line']='海外'
		else:
			print "只能输入 1 2 3"
		for a in add_record_list:
			tdic['sub_domain']=a[0]
			#tdic['record_type']=record_type
			#tdic['record_line_id']='3%3D0'
			#tdic['record_line']='国内'
			tdic['value']=a[1]
			tdic['ttl']=600
			relist.append(json.dumps(tdic,encoding="UTF-8", ensure_ascii=False))
			args = '&records='+str(relist).replace("'","").replace("u{","{")
		#print tdic,'地区:',tdic['record_line']
		print '\n--------------------------------------------------------'
		print args
		raw_input('请确认以上参数，回车继续执行添加')
		res = self.execApi(self.mutirecord,args=args)
		print res['status']['message']

#批量修改记录，多条记录同时修改
	def modifyRecord(self):
		while 1:
			print '\n\n####################################################################################'
			if len(self.record_dic) == 0:
				self.listRecord()
			else:
				self.listRecord(re_search='False')
			choose_type = {
				1: 'sub_domain',
				2: 'record_type',
				3: 'area',
				4: 'value',
				5: 'mx',
				6: 'ttl',
				7: 'status'
			}

			while 1:
				input_change_from = raw_input(
					'1、sub_domain(子域名) 2、record_type（A、CNAME） 3、area（默认、国内、国外） 4、value（解析地址） 5、mx 6、ttl、7、status（enable或者disable），输入编号：')
				if input_change_from.isdigit() and int(input_change_from) in [1, 2, 3, 4, 5, 6, 7]:
					input_change_from = int(input_change_from)
					break

			input_change_to = raw_input('输入目标值，如果修改的是record_type，此项是记录值:')

			if input_change_from == 2:
				input_change_value = raw_input('注意：如果是多条A记录修改为CNAME会删除其他只保留一条，然后修改。(1、CNAME 2、A记录) 输入编号:')
				if input_change_value == '2':
					res = self.exec_Modify('record_type', 'A', input_change_to)
				elif input_change_value == '1':
					for k, v in self.record_dic.items():
						#print k, v
						for x, y in v.items():
							#print "x,y:", x, '###', self.record_dic[k][x]
							if len(y) > 1:
								for times in xrange(len(y) - 1):
									delete_res = self.deleteRecord([k[1], y.pop()])
									print "delete record:", k, x,'--',delete_res['status']['message']
					#print "self.record_dict:", self.record_dic
					modify_res = self.exec_Modify('record_type','CNAME',input_change_to)
				else:
					print "上天不？你想改成啥？"
			else:
				modify_res = self.exec_Modify(choose_type[input_change_from],input_change_to)
#执行修改
	def exec_Modify(self,change_from,change_to,re_value=None):
		self.public_dic["record_id"] = ''
		self.public_dic["change"] = change_from
		self.public_dic["change_to"] = change_to
		for domain,records in self.record_dic.items():
			#print "############################################################################################",domain
			#print records
			if change_from == 'record_type':
				self.public_dic["value"]=re_value
			#self.public_dic["domain"]=domain
			for rec in records:
				#print re
				if 'record_id' in self.public_dic:
					self.public_dic["record_id"] = self.public_dic["record_id"] + ',' + self.record_dic[domain][rec][0]
				else:
					self.public_dic["record_id"] = self.record_dic[domain][rec][0]
		print self.public_dic
		res = self.execApi(self.mutimodify)
		print change_from,"修改结果：",res['status']['message']
		return res

#批量删除记录
	def deleteRecord(self,delete_record=None):
		if delete_record:
			self.public_dic["domain_id"]=delete_record[0]
			self.public_dic["record_id"]=delete_record[1]
			res = self.execApi(self.mutidel)
			
		else:
			self.listRecord()
			#print '删除目标：',self.record_dic
			for k,v in self.record_dic.items():
				for x,y in v.items():
					print '删除目标：',k[0],x
			raw_input("回车继续删除")
			for domain,records in self.record_dic.items():
				self.public_dic["domain_id"]=domain[1]
				print '\n', '''###################################################################################################''', domain[0]
				for record,id_list in records.items():
					for del_id in id_list:
						self.public_dic["record_id"]=del_id
						res = self.execApi(self.mutidel)
						print record,':',del_id,res['status']['message']
		return res

#批量删除域名
	def deleteDomain(self):
		self.listDomain()
		print self.re_list
		raw_input('你最好确认好了！！！！！！！！！！！！！！ 回车确认删除')
		for del_domain_id in self.re_list:
			self.public_dic["domain_id"]=del_domain_id[1]
			res = self.execApi(self.mutidel_domain)
			print res['status']['message']

#获取操作日志	
	def getLog(self):
		self.listDomain()
		for dd in self.re_list:
			self.public_dic["domain"] = dd[0]
			res = self.execApi(self.getlog)
			print '\n', '''###################################################################################################''',dd[0]
			for log in res['log']:
				print log.encode('utf-8')

#获取监控日志	
	def getMonitorlog(self):
		Mlog = open('monitor.log','w')
		self.public_dic["length"] = 2000
		res = self.execApi(self.getmonitorlog)
		print '\n', '''###################################################################################################'''
		for log in res['monitor_downs']:
			if len(log['switch_log']) < 2:
				print log['warn_reason'],'\t\t\t\t\t\t\t\t\t',log['ip'],'\t\t',log['host']
				Mlog.write('%s \t\t\t\t\t\t\t\t\t\t\t  %s \t %s \n' % (log['warn_reason'],log['ip'],log['host']))
			else:
				print log['warn_reason'],'\t',log['switch_log'][0],'\t',log['switch_log'][1],'\t',log['ip'],'\t',log['host']
				Mlog.write('%s \t %s \t %s \t  %s \t %s \n' % (log['warn_reason'],log['switch_log'][0],log['switch_log'][1],log['ip'],log['host']))
		Mlog.close()

        def public_Change(self,**change_dict):
		self.listDomain()
		for sub,domain in change_dict.items():
			#print sub,'11111111111111'
			self.listRecord('True','False',input_search='name='+sub)
			#print self.record_dic
			res = self.exec_Modify('value',domain)
			if res['status']['code'] != '1':
				print "Waiting 3s to tr-try"
				time.sleep(3)
				res = self.exec_Modify('value',domain)
		
			time.sleep(3)
		return self.userGuide()

#以下都是批量修改为固定域名
	def b79_prod_changeTo_block(self):
		self.public_dic["login_token"] = self.choose_account[2]
		change_dict = {'www|@':'block01_web.abc.com','m':'block01_mobile.abc.com','vip':'block01_vipweb.abc.com','vipm':'block01_vipm.abc.com'}
		self.public_Change(**change_dict)

		
	def b79_prod_changeTo_301(self):
		self.public_dic["login_token"] = self.choose_account[2]
		change_dict = {'www|@':'301web.abc.com','m':'301mobile.abc.com','vip':'301vipweb.abc.com','vipm':'301vipm.abc.com'}
		self.public_Change(**change_dict)

	def b79_mkt_changeTo_block(self):
		self.public_dic["login_token"] = self.choose_account[3]
		change_dict = {'www|@':'block01_web.abc.com','m':'block01_mobile.abc.com','vip':'block01_vipweb.abc.com','vipm':'block01_vipm.abc.com'}
		self.public_Change(**change_dict)

	def b79_mkt_changeTo_301(self):
		self.public_dic["login_token"] = self.choose_account[3]
		change_dict = {'www|@':'301web.abc.com','m':'301mobile.abc.com','vip':'301vipweb.abc.com','vipm':'301vipm.abc.com'}
		self.public_Change(**change_dict)

	def e03_changeTo_block(self):
		self.public_dic["login_token"] = self.choose_account[4]
		change_dict = {'www|@':'block01_web.cdnv7.com','m':'block01_mobile.cdnv7.com'}
		self.public_Change(**change_dict)

	def e04_changeTo_block(self):
		self.public_dic["login_token"] = self.choose_account[4]
		change_dict = {'www|@':'block01_web.cdnv8.com','m':'block01_mobile.cdnv8.com'}
		self.public_Change(**change_dict)

	def userGuide(self):
		# os.system('clear')
		self.choose_account = {
			1:'id,key',
			2:'id,key',
			3:'id,key',
			4:'id,key',
			5:'id,key',
			6:self.b79_prod_changeTo_block,
			7:self.b79_prod_changeTo_301,
			8:self.b79_mkt_changeTo_block,
			9:self.b79_mkt_changeTo_301,
			10:self.e03_changeTo_block,
			11:self.e04_changeTo_block
		}
		print '''
				批量操作需谨慎，可以使用测试账号进行测试

				1、测试账号
				2、产品账号
				3、市场账号
				4、NNN 账号
				5、6UP 账号
				6、 产品 批量修改域名到免备案
				7、 产品 批量修改域名到301
				8、 市场 批量修改域名到免备案
				9、 市场 批量修改域名到301
				10、 批量修改域名到免备案
				11、 批量修改域名到免备案
'''
		choose_line = raw_input('输入编号:')
		if  choose_line.isdigit() and 0 < int(choose_line) <= 5:
			choose_line = int(choose_line)
			self.public_dic["login_token"] = self.choose_account[choose_line]
		elif choose_line.isdigit() and 5 < int(choose_line) <= 11:
			self.choose_account[int(choose_line)]()
		else:
			print '您谨慎操作，避免酿成大祸'
			self.userGuide()
		switch = {
			0:self.userGuide,
			1:self.listDomain,
			2:self.listRecord,
			3:self.addMutidomain,
			4:self.addMutirecord,
			5:self.modifyRecord,
			6:self.deleteRecord,
			7:self.deleteDomain,
			8:self.getLog,
			9:self.getMonitorlog,
			111:sys.exit
		}
	
		while 1:
#			try:
				print '''
				选择需要的参数:
						0、重新选择账号						

						1、列出此账号域名、域名搜索 (默认选项，回车继续)
						2、批量查询域名记录（或者包含哪些关键字）
						3、批量增加域名
						4、批量增加记录
						5、批量修改记录（目标域名必须有相同记录）
						6、批量删除记录
						7、批量删除域名
						8、批量获取日志
						9、获取D监控日志
						111、退出
					'''
			
				num = raw_input("选择功能，输入编号:")
				if num:
					if num.isdigit():
						num = int(num)
						if num in [0,1,2,3,4,5,6,7,8,9,10,111]:
							res = switch[num]()
					else:
						print "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!别乱输入，小心爆炸"
					
#			except  Exception,e:
#				print e
#				continue

if __name__ == '__main__':
	p = dnsPodapi()
	p.userGuide()
	#p.listRecord()
```

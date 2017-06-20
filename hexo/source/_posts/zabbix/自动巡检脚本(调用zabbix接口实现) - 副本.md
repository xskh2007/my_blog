---
title: 自动巡检脚本(调用zabbix接口实现)
date: 2017-06-15 14:17:27
categories:	zabbix
tags: 
	- zabbix
---

<!-- toc -->



	crontab: no changes made to crontab
	[root@autoyunwei ~]# vim /etc/crontab 
	[root@autoyunwei ~]# vim /root/script/xunjian.py
	[root@autoyunwei ~]# cat  /root/script/xunjian.py
	#!/usr/bin/env python
	# -*- coding:utf-8 -*- 
	#Author: qiantu
	#qq 261767353
	
	import os
	import time
	import shutil
	import MySQLdb
	import smtplib
	import requests
	import datetime
	
	from email.MIMEText import MIMEText
	from email.MIMEImage import MIMEImage
	from email.MIMEMultipart import MIMEMultipart
	
	# based on zabbix 2.4.4
	ZABBIX_HOST = '192.168.2.91'
	ZABBIX_USER = 'Admin'
	ZABBIX_PWD = 'zzjr#2015'
	
	ZABBIX_DB_HOST = '192.168.2.91'
	ZABBIX_DB_USER = 'zabbix'
	ZABBIX_DB_PWD = 'zabbix'
	ZABBIX_DB_NAME = 'zabbix'
	
	GRAPH_PATH = "zabbix_img"
	GRAPH_PERIOD = 86400  # one day
	
	EMAIL_DOMAIN = 'zuozh.com'
	EMAIL_USERNAME = 'wangqiantu'
	EMAIL_PASSWORD = 'Lht1111'
	
	
	def query_screens(screen_name):
		conn = MySQLdb.connect(host=ZABBIX_DB_HOST, user=ZABBIX_DB_USER, passwd=ZABBIX_DB_PWD,
							db=ZABBIX_DB_NAME, charset='utf8', connect_timeout=20)
		cur = conn.cursor()
		count = cur.execute("""
				select a.name, a.screenid, b.resourceid, b.width, b.height
					from screens a, screens_items as b
					where a.screenid=b.screenid and a.templateid<=>NULL and a.name='%s'
					order by a.screenid;
				""" % screen_name)
		if count == 0:
			result = 0
		else:
			result = cur.fetchall()
	
		cur.close()
		conn.close()
	
		return result
	
	
	def generate_graphs(screens):
		login_resp = requests.post('http://%s/zabbix/index.php' % ZABBIX_HOST, data={
			'name': ZABBIX_USER,
			'password': ZABBIX_PWD,
			'enter': 'Sign in',
			'autologin': 1,
		})
		session_id = login_resp.cookies['zbx_sessionid']
	
		graphs = []
		for i, (screen_name, screen_id, graph_id, width, height) in enumerate(screens):
			params = {
				'screenid': screen_id,
				'graphid': graph_id,
				'width': width * 2,
				'height': height * 2,
				'period': GRAPH_PERIOD,
				'stime': datetime.datetime.utcnow().strftime('%Y%m%d%H%M%S'),
			}
			resp = requests.get('http://%s/zabbix/chart2.php' % ZABBIX_HOST, params=params,
								cookies={'zbx_sessionid': session_id})
			file_name = '_'.join(map(str, screens[i][:3])).replace(' ', '_') + '.png'
			with open(os.path.join(GRAPH_PATH, file_name), 'wb') as fp:
				fp.write(resp.content)
			graphs.append(file_name)
	
		return graphs
	
	
	def send_mail(screen_name, graphs, to_list):
		me = u'巡检报告 <%s@%s>' % (EMAIL_USERNAME, EMAIL_DOMAIN)
	
		def _create_msg():
			msg = MIMEMultipart('related')
			title=u"每日巡检报告"
			msg['Subject'] = '%s: %s' % (title,screen_name)
			msg['From'] = me
			msg['To'] = ';'.join(to_list)
			msg.preamble = 'This is a multi-part message in MIME format.'
	
			contents = "<h1>Screen %s</h1><br>" % screen_name
			contents += "<table>"
			for g_name in graphs:
				with open(os.path.join(GRAPH_PATH, g_name), 'rb') as fp:
					msg_image = MIMEImage(fp.read())
					msg_image.add_header('Content-ID', "<%s>" % g_name)
					msg.attach(msg_image)
	
				contents += ''
				contents += "<tr><td><img src='cid:%s'></td></tr>" % g_name
			contents += "</table>"
	
			msg_text = MIMEText(contents, 'html')
			msg_alternative = MIMEMultipart('alternative')
			msg_alternative.attach(msg_text)
			msg.attach(msg_alternative)
	
			return msg
	
		try:
			server = smtplib.SMTP()
			server.connect('smtp.%s' % EMAIL_DOMAIN)
			server.login('%s@%s' % (EMAIL_USERNAME, EMAIL_DOMAIN), EMAIL_PASSWORD)
			server.sendmail(me, to_list, _create_msg().as_string())
			server.close()
			print 'send mail Ok!'
		except Exception, e:
			print e
	
	
	if __name__ == '__main__':
		# remove old dirs
		if os.path.exists(GRAPH_PATH):
			shutil.rmtree(GRAPH_PATH)
		os.makedirs(GRAPH_PATH)
	
		for srn_name in (u'All databases machine cpu',u'All virtual machine io',u'All virtual machine memory',u'All virtual machine traffic',u'ALL nginx connections'):
			# get screens
			all_screens = query_screens(srn_name)
			print all_screens
	
			# generate graphs
			graphs = generate_graphs(all_screens)
	
			send_mail(srn_name, graphs, ['monitor@zuozh.com'])
			#send_mail(srn_name, graphs, ['wangqiantu@zuozh.com'])
	
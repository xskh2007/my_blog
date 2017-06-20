---
title: zabbix批量创建web_scenario
date: 2017-06-19 14:17:27
categories:	zabbix
tags: 
	- zabbix
---

<!-- toc -->



	#!/usr/bin/env python
	# -*- coding:utf-8 -*- 
	#Author: qiantu
	#qq 261767353
	import urllib2
	import json
	
	zabbixurl = 'http://192.168.2.91/zabbix/api_jsonrpc.php'
	username = "Admin"
	password = "zzjr#2015"
	
	
	
	'''登陆函数获取登录auth'''
	def authenticate(zabbixurl, username, password):
		values = {'jsonrpc': '2.0',
				'method': 'user.login',
				'params': {
					'user': username,
					'password': password
				},
				'id': '0'
				}
	
		data = json.dumps(values)
		req = urllib2.Request(zabbixurl, data, {'Content-Type': 'application/json-rpc'})
		response = urllib2.urlopen(req, data)
		output = json.loads(response.read())
	
	
		try:
			message = output['result']
	
		except:
			message = output['error']['data']
			quit()
	
		return output['result']
	
	auth=authenticate(zabbixurl, username, password)
	print auth
	
	'''根据主机名获取hostid'''
	def gethost():
		values={
		"jsonrpc": "2.0",
		"method": "host.get",
		"params": {
			"output": "extend",
			"filter": {
				"host": [
					"Zabbix server"
				]
			}
		},
		"auth": auth,
		"id": 1
	}
		data = json.dumps(values)
		req = urllib2.Request(zabbixurl, data, {'Content-Type': 'application/json-rpc'})
		response = urllib2.urlopen(req, data)
		host_get = response.read()
		host=json.loads(host_get)
		return host
	
	hostid=gethost()['result'][0]['hostid']
	print hostid
	
	'''创建web监控'''
	def create_web_scenario(name,url):
			values={"jsonrpc": "2.0",
					"method": "httptest.create",
					"params": {
						"name": name,
						"hostid": hostid,
						"steps": [
							{
								"name": name,
								"url": url,
								"status_codes": 200,
								"no": 1
							}
						]
					},
					"auth": auth,
					"id": 1
					}
			data = json.dumps(values)
			print url
			req = urllib2.Request(zabbixurl, data, {'Content-Type': 'application/json-rpc'})
			response = urllib2.urlopen(req, data)
			# print response.read()
			return response.read()
	
	
	# name='api.niubangold.com/api/v2/member/assets'
	# print create_web_scenario(name,url)
	
	
	if __name__=='__main__':
		webapi_urls=open('url.txt',mode='r').readlines()
		for webapi_url in webapi_urls:
			url=webapi_url.strip()
			name=url[7:].split('/')[1:]
			name='/'.join(name)
			# print name
	print create_web_scenario(name,url)
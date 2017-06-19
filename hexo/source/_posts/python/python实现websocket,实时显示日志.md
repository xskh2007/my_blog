---
title: 使用python简单的实现websocket服务器
date: 2017-06-15 14:17:27
categories:	websocket
tags: 
	- websocket
---

<!-- toc -->



#### 一、开始的话
使用python简单的实现websocket服务器，可以在浏览器上实时显示远程服务器的日志信息。
之前做了一个web版的发布系统，但没实现在线看日志，每次发布版本后，都需要登录到服务器上查看日志，非常麻烦，为了偷懒，能在页面点几下按钮完成工作，所以这几天查找了这方面的资料，实现了这个功能，瞬间觉的看日志什么的，太方便了，以后也可以给开发们查日志，再也不用麻烦运维了，废话少说，先看效果吧。
[图片]
#### 二、代码
在实现这功能前，看过别人的代码，发现很多都是只能在web上显示本地的日志，不能看远程主机上的日志，有些能看远程日志的是引用了其他框架（例如bottle，tornado）来实现的，而且所有这些都是重写thread的run方法来实现的，由于本人技术太菜，不知道怎么改成自己需要的样子，而且我是用django这个web框架的，不想引用太多框架，搞的太复杂，所以用python来实现websocket服务器。由于技术问题，代码有点粗糙，不过能实现功能就行，先将就着用吧。
执行下面命令启动django和websocketserver
	nohup python manage.py runserver 10.1.12.110 &
	nohup python websocketserver.py &

#### 启动websocket后，接收到请求，起一个线程和客户端握手，然后根据客户端发送的ip和type，去数据库查找对应的日志路径，用paramiko模块ssh登录到远程服务器上tail查看日志，再推送给浏览器，服务端完整代码如下：
	
	# coding:utf-8
	import struct
	import base64
	import hashlib
	import socket
	import threading
	from public.public import get_ssh
	import os
	
	
	def recv_data(conn):    # 服务器解析浏览器发送的信息
		try:
			all_data = conn.recv(1024)
			if not len(all_data):
				return False
		except:
			pass
		else:
			code_len = ord(all_data[1]) & 127
			if code_len == 126:
				masks = all_data[4:8]
				data = all_data[8:]
			elif code_len == 127:
				masks = all_data[10:14]
				data = all_data[14:]
			else:
				masks = all_data[2:6]
				data = all_data[6:]
			raw_str = ""
			i = 0
			for d in data:
				raw_str += chr(ord(d) ^ ord(masks[i % 4]))
				i += 1
			return raw_str
	
	
	def send_data(conn, data):   # 服务器处理发送给浏览器的信息
		if data:
			data = str(data)
		else:
			return False
		token = "\x81"
		length = len(data)
		if length < 126:
			token += struct.pack("B", length)    # struct为Python中处理二进制数的模块，二进制流为C，或网络流的形式。
		elif length <= 0xFFFF:
			token += struct.pack("!BH", 126, length)
		else:
			token += struct.pack("!BQ", 127, length)
		data = '%s%s' % (token, data)
		conn.send(data)
		return True
	
	
	def handshake(conn, address, thread_name):
		headers = {}
		shake = conn.recv(1024)
		if not len(shake):
			return False
	
		print ('%s : Socket start handshaken with %s:%s' % (thread_name, address[0], address[1]))
		header, data = shake.split('\r\n\r\n', 1)
		for line in header.split('\r\n')[1:]:
			key, value = line.split(': ', 1)
			headers[key] = value
	
		if 'Sec-WebSocket-Key' not in headers:
			print ('%s : This socket is not websocket, client close.' % thread_name)
			conn.close()
			return False
	
		MAGIC_STRING = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11'
		HANDSHAKE_STRING = "HTTP/1.1 101 Switching Protocols\r\n" \
						"Upgrade:websocket\r\n" \
						"Connection: Upgrade\r\n" \
						"Sec-WebSocket-Accept: {1}\r\n" \
						"WebSocket-Origin: {2}\r\n" \
						"WebSocket-Location: ws://{3}/\r\n\r\n"
	
		sec_key = headers['Sec-WebSocket-Key']
		res_key = base64.b64encode(hashlib.sha1(sec_key + MAGIC_STRING).digest())
		str_handshake = HANDSHAKE_STRING.replace('{1}', res_key).replace('{2}', headers['Origin']).replace('{3}', headers['Host'])
		conn.send(str_handshake)
		print ('%s : Socket handshaken with %s:%s success' % (thread_name, address[0], address[1]))
		print 'Start transmitting data...'
		print '- - - - - - - - - - - - - - - - - - - - - - - - - - - - - -'
		return True
	
	
	def dojob(conn, address, thread_name):
		from mode import models
		handshake(conn, address, thread_name)     # 握手
		log_info = recv_data(conn)
		log_ip = log_info.split(":")[0]
		log_type = log_info.split(":")[1]
	
		auth_ = models.Authentication.objects.get(ip=log_ip)
		user = auth_.a_user
		pwd = auth_.a_password
		try:
			log_path = models.LogPath.objects.get(ip=log_ip, type_name__contains=log_type).log_path
		except Exception, e:
			send_data(conn, e)
			conn.close()
			print "Error:" + str(e)
			print ('%s : Socket close with %s:%s' % (thread_name, address[0], address[1]))
			return
		conn.setblocking(0)                       # 设置socket为非阻塞
		ssh = get_ssh(log_ip, user, pwd)
		ssh_t = ssh.get_transport()
		chan = ssh_t.open_session()
		chan.setblocking(0)   # 设置非阻塞
		chan.exec_command('tail -f %s' % log_path)
		while True:
			clientdata = recv_data(conn)
			if clientdata is not None and 'quit' in clientdata:
				print ('%s : Socket close with %s:%s' % (thread_name, address[0], address[1]))
				send_data(conn, 'close connect')
				conn.close()
				break
			while True:
				while chan.recv_ready():
					clientdata1 = recv_data(conn)
					if clientdata1 is not None and 'quit' in clientdata1:
						print ('%s : Socket close with %s:%s' % (thread_name, address[0], address[1]))
						send_data(conn, 'close connect')
						conn.close()
						break
					log_msg = chan.recv(10000).strip()
					print log_msg
					send_data(conn, log_msg)
				if chan.exit_status_ready():
					break
				clientdata2 = recv_data(conn)
				if clientdata2 is not None and 'quit' in clientdata2:
					print ('%s : Socket close with %s:%s' % (thread_name, address[0], address[1]))
					send_data(conn, 'close connect')
					conn.close()
					break
			break
	
	
	def ws_service():
	
		os.environ.setdefault("DJANGO_SETTINGS_MODULE", "lbg.settings")
		index = 1
		sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		sock.bind(("127.0.0.1", 12345))
		sock.listen(100)
	
		print ('\r\n\r\nWebsocket server start, wait for connect!')
		print '- - - - - - - - - - - - - - - - - - - - - - - - - - - - - -'
		while True:
			connection, address = sock.accept()
			thread_name = 'thread_%s' % index
			print ('%s : Connection from %s:%s' % (thread_name, address[0], address[1]))
			t = threading.Thread(target=dojob, args=(connection, address, thread_name))
			t.start()
			index += 1
	
	
	ws_service()

#### get_ssh的代码如下：

	import paramiko
	def get_ssh(ip, user, pwd):
		try:
			ssh = paramiko.SSHClient()
			ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
			ssh.connect(ip, 22, user, pwd, timeout=15)
			return ssh
		except Exception, e:
			print e
			return "False"

#### 打开页面时，自动连接websocket服务器，完成握手，并发送ip和type给服务端，所以可以看不同类型，不同机器上的日志，
图片

#### 页面代码如下:
	<!DOCTYPE html>  
	<html>  
	<head>
	{% include 'head_script.html' %}
	<style>
		#log {
			width: 440px;
			height: 200px;
			border: 1px solid #7F9DB9;
			overflow: auto;
		}
		pre {
			margin: 0 0 0;
			padding: 0;
			border: hidden;
			background-color: #0c0c0c;
			color: #00ff00;
		}
		#btns {
			text-align: right;   {# 按钮靠右 #}
		}
	</style>
		<script>
			var socket;
			function init() {  
				var host = "ws://127.0.0.1:12345/";
				var ips="{{ ip }}";
				var types="{{ type }}";
				var log_msg = ips +":"+ types;
	
				console.log(log_msg);
				try {
					socket = new WebSocket(host);  
					socket.onopen = function () {
						log('Connected');
						socket.send(log_msg);
					};  
					socket.onmessage = function (msg) {  
						log(msg.data);
						var obje = document.getElementById("log");   //日志过多时清屏
						var textlength = obje.scrollHeight;
						if (textlength > 10000) {
							obje.innerHTML = '';
						}
					};  
					socket.onclose = function () {
						log("Lose Connection!");
						$("#start").attr('disabled', false);
						$("#stop").attr('disabled', true);
					};
					$("#start").attr('disabled', true);
					$("#stop").attr('disabled', false);
				}
				catch (ex) {  
					log(ex);  
				}
			}
			window.onbeforeunload = function () {  
				try {  
					socket.send('quit');  
					socket.close();  
					socket = null;  
				}  
				catch (ex) {  
					log(ex);  
				}  
			};
			function log(msg) {
				var obje = document.getElementById("log");
				obje.innerHTML += '<pre><code>' + msg + '</code></pre>';
				obje.scrollTop = obje.scrollHeight;   //滚动条显示最新数据
			}
			function stop() {
				try {
					log('Close connection!');
					socket.send('quit');
					socket.close();
					socket = null;
					$("#start").attr('disabled', false);
					$("#stop").attr('disabled', true);
				}
				catch (ex) {
					log(ex);
				}
			}
			function closelayer() {
				try {
					log('Close connection!');
					socket.send('quit');
					socket.close();
					socket = null;
				}
				catch (ex) {
					log(ex);
				}
				var index = parent.layer.getFrameIndex(window.name); //先得到当前iframe层的索引
				parent.layer.close(index); //再执行关闭
			}
		</script>
	</head>  
	
	<body onload="init()">  
		<div class="row">
			<div class="col-lg-12">
				<div id="log" style="width: 100%;height:440px;background-color: #0c0c0c;overflow:scroll;overflow-x: auto;"></div>
				<br>
			</div>
		</div>
		<div class="row">
			<div class="col-lg-12">
				<div id="btns">
					<input disabled="disabled" type="button" class="btn btn-primary btn-sm" value="start" id="start" onclick="init()">
					<input disabled="disabled" type="button" class="btn btn-primary btn-sm" value="stop" id="stop" onclick="stop()" >
					<input type="button" class="btn btn-primary btn-sm" value="close" id="close" onclick="closelayer()" >
				</div>
			</div>
		</div>
	</body>  
	{% include 'foot_script.html' %}
	</html> 
	
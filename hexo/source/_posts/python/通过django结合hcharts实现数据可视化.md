---
title: 通过django结合hcharts实现数据可视化
date: 2017-06-18 19:17:27
categories:	hcharts
tags: 
	- hcharts
---

<!-- toc -->



    我们都知道现在到处都是轮子，但是还想自己去造一个属于自己的轮子，所以自己在学习django的同时也尝试模仿着别人造轮子。django怎么样就不说了，让别人度娘去吧；

通过django结合hcharts动态展现内存使用率，实现单机内存监控效果，因为度娘里的echarts文档相对难看懂，觉得hcharts实现比较简单。
 

本文基于Centos7下虚拟环境安装：Python3.5.2实现；

Ip地址：10.0.10.250

运行环境：
 

    (py35env) [root@jupyter memory]# pip list 
    appdirs (1.4.3) 
    decorator (4.0.11) 
    Django (1.11.1) 
    ipython (6.0.0) 
    ipython-genutils (0.2.0) 
    jedi (0.10.2) 
    packaging (16.8) 
    pexpect (4.2.1) 
    pickleshare (0.7.4) 
    pip (9.0.1) 
    prompt-toolkit (1.0.14) 
    ptyprocess (0.5.1) 
    Pygments (2.2.0) 
    PyMySQL (0.7.11)    #python3使用mysql的工具包变成了这个名字； 
    pyparsing (2.2.0) 
    pytz (2017.2) 
    requests (2.14.2)    
    setuptools (35.0.2)     #安装PyMySQL前先yum install setuptool -y 
    simplegeneric (0.8.1) 
    six (1.10.0) 
    traitlets (4.3.2) 
    wcwidth (0.1.7) 
    wheel (0.29.0) 

 

其实很多实验之所以会失败大部分都是软件的版本和兼容性不同导致，本文是参考了51reboot的内存可视化演变而来，只是我使用了自己最近学习的django来实现前端展现。

1、创建一个project：
 

    django-admin startproject memory 

2、进入memory目录下并创建一个app：
 

    cd memory && django-admin startapp blog 

3、创建两个目录，static和templates，一个存放静态页面和静态文件如js、css；
 

    mkdir static templates 

查看目录结构：
 

    (py35env) [root@jupyter ~]# tree -d memory/ -L 1 
    memory/ 
    ├── blog 
    ├── memory 
    ├── static 
    └── templates 

 

使用这个图形，并把html代码里的4个js文件下载到static/js下：https://www.hcharts.cn/demo/highstock/spline

jquery-1.8.3.min.js、highstock.js、exporting.js、highcharts-zh_CN.js

 

4、接下来配置一下memory/setting.py：
 

    28 ALLOWED_HOSTS = ['*']            #改成'*'                                                 
     
    55 TEMPLATES = [                                                                     
    56     {                                                                             
    57         'BACKEND': 'django.template.backends.django.DjangoTemplates',             
    58         'DIRS': [                                                                 
    59             os.path.join(BASE_DIR, 'templates'),        #加入这一行，html文件的目录                          
    60         ],                                                                        
    61         'APP_DIRS': True,                                                         
    62         'OPTIONS': {                                                              
    63             'context_processors': [                                               
    64                 'django.template.context_processors.debug',                       
    65                 'django.template.context_processors.request',                     
    66                 'django.contrib.auth.context_processors.auth',                    
    67                 'django.contrib.messages.context_processors.messages',            
    68             ],                                                                    
    69         },                                                                        
    70     },                                                                            
    71 ] 

 

最后面添加，静态js和CSS文件的存放目录：
 

    124 STATICFILES_DIRS = (                                                                                                                                     
    125     os.path.join(BASE_DIR, 'static'),                                             
    126 )  

5、配置blog/views.py文件：

    #!/usr/bin/env python3                                                               
    # -*- coding:UTF-8 -*-                                                               
    from django.shortcuts import render                                                  
    from django.http import HttpResponse                                                 
    import pymysql as mysql                                                              
    import json                                                                          
    import requests                                                                      
                                                                                         
    # Create your views here.                                                            
    tmp_time = 0                                                                         
    con = mysql.connect(user='dba',passwd='123456',host='localhost',db='memory')         
    con.autocommit(True)                                                                 
    cur = con.cursor()                                                                   
                                                                                         
    def index(request):                                                                  
        return render(request, 'demo.html')                                              
                                                                                         
    def data(request):                                                                                                                                            
        global tmp_time                                                                  
        if tmp_time>0:                                                                   
            sql = 'select * from memory where time>%s' %(tmp_time/1000)                  
        else:                                                                            
            sql = 'select * from memory'                                                 
        cur.execute(sql)                                                                 
        arr = []                                                                         
        for i in cur.fetchall():                                                         
            arr.append([i[1]*1000,i[0]])                                                 
        if len(arr)>0:                                                                   
            tmp_time = arr[-1][0]                                                        
        return HttpResponse(json.dumps(arr))  

	

6、配置url.py文件：

    16 from django.conf.urls import url                                                  
     17 from django.contrib import admin                                                  
     18 from blog import views as blog_views          #引入blog里的views文件，并定义一个别名；                                    
     19                                                                                   
     20 urlpatterns = [                                                                   
     21     url(r'^admin/', admin.site.urls),                                             
     22     url(r'^$', blog_views.index),      #以下两行为加入内容                                           
     23     url(r'^data/',blog_views.data),                                                                                                                          
     24 ]         

 

7、 接下来创建一个静态html文件，也就是views.py里定义的demo.html文件：
(py35env) [root@jupyter memory]# vim templates/demo.html 
 
 

    {% load static %}       #引入static目录下的静态文件要先加入此行 
    <!DOCTYPE html> 
    <html lang="en"> 
    <head> 
        <meta charset="UTF-8"> 
        <title>内存监控信息</title> 
        <script src="{% static 'js/jquery-1.8.3.min.js' %}"></script> 
        <script src="{% static 'js/highstock.js' %}"></script> 
        <script src="{% static 'js/exporting.js' %}"></script> 
        <script src="{% static 'js/highcharts-zh_CN.js' %}"></script> 
    </head> 
    <body> 
        <div id="container" style="min-width:400px; height:400px"></div> 
        <script> 
                $(function () { 
                $.getJSON('/data', function (data) { 
                    // Create the chart 
                    $('#container').highcharts('StockChart', { 
                        chart:{ 
                            events:{ 
                                load:function(){ 
                                    var series = this.series[0] 
                                    setInterval(function(){ 
                                    $.getJSON('/data',function(res){ 
                                        $.each(res,function(i,v) { 
                                            series.addPoint(v) 
                                        }) 
                                    }) 
                                    },3000) 
                                } 
                            } 
                        }, 
                        rangeSelector: { 
                            selected: 1 
                        }, 
                        title: { 
                            text: '内存使用率-单位M' 
                        }, 
                        series: [{ 
                            name: '内存使用率', 
                            data: data, 
                            type: 'spline', 
                            tooltip: { 
                                valueDecimals: 2 
                            } 
                        }] 
                    }); 
                }); 
            }); 
        </script> 
    </body> 
    </html> 

 

其实仔细看跟hcharts上面的实例差不多，只是加入了几行代码实现动态刷新；
 

    chart:{ 
        events:{ 
            load:function(){ 
                var series = this.series[0] 
                setInterval(function(){ 
                $.getJSON('/data',function(res){ 
                    $.each(res,function(i,v) { 
                        series.addPoint(v) 
                    }) 
                }) 
                },3000) 
            } 
        } 
    }, 

 

8、接下来启动django开发环境；
 

    cd memory 
    (py35env) [root@jupyter memory]# python manage.py runserver 0.0.0.0:8000 
    Performing system checks... 
     
    System check identified no issues (0 silenced). 
    May 26, 2017 - 11:02:18 
    Django version 1.11.1, using settings 'memory.settings' 
    Starting development server at http://0.0.0.0:8000/ 
    Quit the server with CONTROL-C. 

 
然后通过浏览器就可以访问10.0.10.250:8000看到效果了；
![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/anything/1_111719_1.jpg)
 
这里调用的数据是上一篇 python 获取centos7的内存使用率并写入mysql   ，只要把monitor.py执行就可以看到动态的展现效果了。

 

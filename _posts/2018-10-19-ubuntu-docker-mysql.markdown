---
layout: post
title: ubuntu16.04 docker install MySQL
date: 2018-10-19
description: # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [MYSQL]
---

1.安装
{% highlight ruby %} 
	sudo docker pull docker.io/mysql
{% endhighlight %} 

2.启动并讲数据挂载在宿主机上
{% highlight ruby %}
	sudo docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v /home/mysql/data:/var/lib/mysql -v /home/mysql/conf.d:/etc/mysql/conf.d -d mysql

	--name：容器名

	--p：映射宿主主机端口

	-v：挂载宿主目录到容器目录

	-e：设置环境变量，此处指定root密码

	-d：后台运行容器
{% endhighlight %}

3.进入容器
{% highlight ruby %}
	sudo docker exec -it mysql /bin/bash
{% endhighlight %}

4.进入MySQL客户端
{% highlight ruby %}
	mysql -uroot -p
{% endhighlight %}

done~

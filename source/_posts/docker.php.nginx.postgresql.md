---
layout: post
title: docker + php + nginx + postgresql
---
1. 启动容器  
	```Bash
	$ docker run --name gg.php -p 2120:2120 -p 2121:2121 -p 2123:2123 -d \
	-v /volume1/homes/ggzdttt/app/php:/var/www/html \
	php:7.1-fpm
	```

	如果步骤2和3都成功后执行下面的命令(记得将时间更改为北京时) 
 
	```Bash
	$ docker exec -it gg.php /bin/bash
	$ cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
	$ php /var/www/html/gzt/server/other/serial/realtime/start.php start -d
	$ exit
	
	$ docker run --name gg.nginx -p 8765:80 -d \
	-v /volume1/homes/ggzdttt/app/php:/usr/share/nginx/html \
	-v /volume1/homes/ggzdttt/nginx/conf.d:/etc/nginx/conf.d \
	--link gg.php:php \
	nginx
	
	$ docker exec -it gg.nginx /bin/bash
	$ cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
	$ exit
	```

2. 安装php关于postgresql数据库的依赖  
	```Bash
	$ docker exec -it gg.php /bin/bash
	$ apt-get update
	$ apt-get install -y libpq-dev
	$ /usr/local/bin/docker-php-ext-configure pgsql
	$ /usr/local/bin/docker-php-ext-install pgsql
	$ exit
	$ docker restart gg.php
	```

3. 安装workerman依赖  
	```Bash
	$ docker exec -it gg.php /bin/bash
	$ apt-get install -y wget
	$ #执行`php -v`确定php版本，然后到http://php.net/releases/查找相应版本的php
	$ wget http://php.net/distributions/php-7.1.4.tar.gz
	$ tar zxvf php-7.1.4.tar.gz
	$ cd php-7.1.4/ext/pcntl/
	$ phpize
	$ ./configure
	$ make
	$ make install
	$ #通过运行 php --ini查找php.ini文件位置，然后在文件中添加extension=pcntl.so
	$ apt-get install -y vim
	$ vim /usr/local/etc/php/conf.d/docker-php-ext-pcntl.ini
	$ #在文件中添加extension=pcntl.so，最后保存退出
	$ php /var/www/html/gzt/server/other/serial/realtime/start.php start -d
	$ exit
	$ docker restart gg.php
	```
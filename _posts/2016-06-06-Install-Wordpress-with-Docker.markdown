---
layout: post
title:  "Install WordPress with Docker"
date:   2016-06-06 18:00:00 +0700
categories: DevOps
tags:
  - WordPress
  - DevOps
---
จริงๆ แล้วมีหลายวิธีที่สามารถใช้ WordPress บน Docker ซึ่งทาง Docker เองได้เตรียม [Quick start WordPress](https://docs.docker.com/compose/wordpress/) ไว้เหมือนกัน แต่ผมอยากลองทำแบบบ้านๆ ดู เพื่อที่จะไดศึกษาอะไรบ้างนิดหน่อย

ผมไม่ขอพูดถึง DevOps หรือ Docker คืออะไรนะครับ หาอ่านได้จาก [Docker](https://www.docker.com/) ได้เลย

WordPress เป็น PHP ดังนั้นเริ่มจาก LEMP Stack กันก่อน ง่ายๆเลยคือเราต้องมี Linux, Nginx, MySQL, PHP ก่อน ซึ่งถ้าเราสนใจเรื่อง DevOps เราจะมองมันเป็น Microservices แยกออกเป็น Service ไป

มาลองใช้ Nginx บน Docker กัน เริ่มจาก ทำ index.html ขึ้นมาก่อน
{% highlight html %}
<h1>Hello world from Docker!</h1>
{% endhighlight %}

ทดลองรัน Nginx ด้วย Docker
{% highlight shell %}
$ docker run -v `pwd`:/usr/share/nginx/html -p 8080:80 nginx
{% endhighlight %}

ลองตรงสอบผลลัพธ์ด้วย `curl` หรือจะเปิดผ่าน browser ดูก็ได้
สำหรับเครื่อง Mac ต้องหา ip ก่อนจาก `docker-machine ip default`
{% highlight shell %}
$ curl http://192.168.99.100:8080
<h1>Hello world from Docker!</h1>
{% endhighlight %}

โอเค! เป็นอันว่าใช่ได้

ต่อไปเรามาทำ Dockerfile กัน เพื่อที่เราจะ build image เก็บไว้ใช้ในครั้งต่อไป ใน Dockerfile ผมจะใช้ image ที่ต่างไปจากข้างบนนิดหน่อยคือ จะให้ตัว Alpine เหตุผลเพราะว่ามันเล็กและเบากว่า Alpine ทำมาจาก Scratch ซึ่งเป็น empty image ที่ทาง Docker ได้เตรียมไว้ให้สำหรับนักพัฒนา ถ้า Alpine มันดูต้องทำเองเยอะไปเราสามารถใช้ ubuntu เป็น base image ก็ได้เช่นกัน แต่มันจะใหญ่ไปหน่อย เท่านั้นเอง

{% highlight shell %}
FROM nginx:alpine
MAINTAINER Blythe LilYoojun

RUN mkdir -p /var/www/html
WORKDIR /var/www/html

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY ./www ./

CMD ["nginx", "-g", "daemon off;"]
{% endhighlight %}

นี้เป็น Dockerfile ที่เราจะนำไปสร้าง my-wordpress image ตรงนี้มี 2 จุดที่น่าสนใจคือ `COPY nginx.conf /etc/nginx/conf.d/default.conf` คือเป็นการนำ nginx.conf ที่เราเขียนขึ้นมาใหม่ไปใช้ใน container อีกจุดคือ `COPY ./www ./` เป็น WORKDIR ฝั่ง Dev ที่เราจะ mount มันเข้ากับ container ตรงนี้เป็นพื้นฐานเลย คือเราก็ Dev บนเครื่องเราไป แต่เราจะไปให้ Enviroment แทน

เรามาดู nginx.conf กันบ้างดีกว่า
{% highlight bash %}
server {
  server_name _;
  listen 80 default_server;

  root /var/www/html;
  index index.php index.html;

  access_log /dev/stdout;
  error_log /dev/stdout info;

  location / {
    try_files $uri $uri/ /index.php?$args;
  }

  location ~ .php$ {
    include fastcgi_params;
    fastcgi_pass my-php:9000;
    fastcgi_index indux.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
  }
}

{% endhighlight %}

Nginx Apline ไม่ได้มาพร้อมกับ PHP แน่นอน เราตั้งจะใจจะให้มันเป็น Microservices อยู่แล้ว ดั้งนั้น เราต้องสร้าง Dockerfile.php-fpm สำหรับ my-php image อีกตัวหนึ่ง

{% highlight shell %}
FROM php:fpm-alpine
MAINTAINER Blythe LilYoojun

RUN docker-php-ext-install -j$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) iconv gd mbstring fileinfo curl xmlreader xmlwriter spl ftp mysqli

VOLUME /var/www/html
{% endhighlight %}

ต่อไป ผมจะรวมทั้ง 2 อันเข้าด้วยกันด้วย Docker-compose ซึ่งทำให้เราสามารถสั่ง start docker หลายๆ ตัวขึ้นมาพร้อมกันได้ ด้วยการ config ลงไปในไฟล์ docker-compose.yml เรามาดูกัน เพื่อไม่ใช้มันยาวเกินไปในที่นี้ผมจะเพื่อ mysql ลงไปในไฟล์เลยทีเดียว

{% highlight bash %}
version: '2'
services:
  my-wordpress:
    build: .
    volumes:
      - ./www:/var/www/html
    ports:
      - "8080:80"
    links:
      - my-php
  my-php:
    build:
      context: .
      dockerfile: Dockerfile.php-fpm
    volumes:
      - ./www:/var/www/html
    ports:
      - "9000:9000"
    links:
      - my-mysql
  my-mysql:
    image: mariadb
    volumes:
      - /var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: wp
      MYSQL_DATABASE: wp
      MYSQL_USER: wp
      MYSQL_PASSWORD: wp
{% endhighlight %}

`links` ในที่นี้คือเป็นการเชื่อมกันระหว่าง container กับ container
เตรียมทุกอย่างเรียบร้อยแล้ว เราก็สั่งรันได้เลย

{% highlight shell %}
$ docker-compose up
{% endhighlight %}

ถ้าทุกอย่างถูก start ขึ้นมาแล้ว ขั้นตอนต่อไปคือ download WordPress มาใช้งาน เปิด Terminal มาอีกอันแล้วไปใช้ `www` ที่เรา map ไว้ต้องแต่แรก แล้ว download WordPress ลงมา

{% highlight shell %}
$ cd www
$ wget https://wordpress.org/latest.zip
$ unzip latest.zip
{% endhighlight %}

เท่านี้เราก็จะได้ WordPress มาใช้งานบน Docker แล้ว :)


Credit: [WordPress developer’s intro to Docker](https://codeable.io/wordpress-developers-intro-docker/)

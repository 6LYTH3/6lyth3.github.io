---
layout: post
title:  "List All Network Connection"
date:   2016-06-01 14:56:00 +0700
categories: Linux
tags: Linux
---
สวัสดีครับ เราจะรู้ได้อย่างไรว่า เครื่องเราเปิด Service อะไรให้คนอื่นเข้ามาใช้งานบ้าง และมีใครบ้างกำลังเชื่อมต่อกับเครื่องของเราอยู่ Linux ได้เตรียมคำสั่งพื้นฐานในการตรวจเช็ค connection ต่างๆ ไว้ให้เราแล้ว

`lsof` คือ List open files นอกจากจะให้ดู files ที่เปิดอยู่แล้ว ยังสามารถดูว่าใครคน connected มาทำเครื่องเราบ้างได้อีกด้วย

{% highlight shell %}
COMMAND   PID   USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
UserEvent 251 blythe    4u  IPv4 0x981d74149abd0101      0t0  UDP *:*
Google    259 blythe  157u  IPv4 0x981d74149c2bb251      0t0  TCP 192.168.1.40:51002->stackoverflow.com:https (ESTABLISHED)
Google    259 blythe  159u  IPv4 0x981d7414a120d441      0t0  TCP 192.168.1.40:51173->r-199-59-148-84.twttr.com:https (ESTABLISHED)
{% endhighlight %}

`netstat` ใช้สำหรับ show network status แน่นอนว่าน่าจะใช้งานกันทุกคน แต่ที่ชอบที่สุดคือมี option `-p` ที่จะโชว์รายละเอียดของ PID/Program ที่เปิดไว้ ซึ่งจะได้รู้ว่า port นี้เปิดมาจาก program อะไร

{% highlight shell %}
netstat -nlpt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.1.1:53            0.0.0.0:*               LISTEN      1144/dnsmasq    
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      661/cupsd       
tcp6       0      0 ::1:631                 :::*                    LISTEN      661/cupsd
{% endhighlight %}

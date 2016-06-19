---
layout: post
title:  "Hack Wifi with Aircrack"
date:   2016-06-19 16:00:00 +0700
categories: Hacking
tags:
- Wifi
- Hacking
---
มาลอง Hack Wifi กันดูบ้าง ซึ่งจริงๆ แล้วมีหลายวิธีด้วยกัน ขึ้นอยู่กับการตั้งค่าของ Access point หรืออาจจะเป็น Net หอพักที่มีระบบ Login

ที่จะลอง Hack ครั้งนี้เป็น Net บ้านธรรมดา ก็จะมี WPA WPA2 และก็บอก เปิด WPS เอาไว้ให้คนอื่นมา pair แต่ที่หน้าเสียดายคือ แถวบ้านไม่มีใครใช้ WPS เลย อดใช้ `reaver` ต้องไป brute force แทน น่าเศร้าจริงๆ

เริ่มกันเลยดีกว่า สิ่งที่ต้องมีคือ

- Kali Linux
- Wireless Adapter [TP-Link TL-WN722N]
- Access point ชาวบ้าน

เริ่งจากทำ interface ให้เป็น monitor กันก่อนง่ายด้วย
![Airodump Start](/images/airodump-start.png)
ได้แล้ว Kill PID ที่ show ให้หมด ถ้าไม่ show monitor mode แสดงว่า Adapter ใช้งานไม่ได้กับ `Airmon-ng` ซึ่งของผมจะเป็น wlan0mon

หา Access Point เป้าหมายของเรา
{% highlight shell %}
$ airodump-ng wlan0mon
{% endhighlight %}
![Airodump Start](/images/Sniff.png)

เป้าหมายผมคือ `MIW` เริ่มเก็บ Package ด้วย
{% highlight shell %}
$ airodump-ng -c 6 -w miw --bssid D8:97:BA:01:56:B2 wlan0mon
{% endhighlight %}

เปิดอีก Terminal, เพื่อที่จะได้ WPA handshake มา เราต้องมา deauth กับ Client ที่เกาะอยู่ก่อน
{% highlight shell %}
$ aireplay-ng --deauth 0 -a D8:97:BA:01:56:B2 wlan0mon
{% endhighlight %}
![Airodump Start](/images/replay.png)

รอจนกว่าจะได้ WPA handshake มา พอได้มาแล้วก็ `Ctrl-c` ได้เลย

ต่อมาคือการ brute force ซึ่งเป็นเรื่องน่าเศร้ามากสำหรับผม มันใช้เวลานานไป
ถึงจุดนี้เราต้องคิดอย่างที่ user คิดให้ได้
การตั้ง Password ส่วนใหญ่จะเริ่มตั้งแต่ 8 ตัวขึ้นไป และอีกอย่างที่นิยมตั้งกันคือ เบอร์โทรศัพท์ซึ่งมีอยู่ 10 ตัวและเป็นตัวเลขหมด

มีวิธีที่จะ brute force หลายอย่าง ไม่ว่าจะเป็น wordlist ที่เตรียมเอาไว้ หรือจะใช้ crunch ที่ generate word ขึ้นมา ในที่นี้ผมของใช้ crunch ละกัน
![Airodump Start](/images/deauth.png)

หลังจากลอง brute force บน VM แล้วไม่ช้าไปและ Key แรกก็ไม่ตรงเลยด้วย จึงเอาไฟล์ `cap` ออกมา brute force ที่เครื่องอื่นและเพิ่ม key ที่จะใช้หาเข้าไปด้วย
![Airodump Start](/images/findky.png)

รอ รอ แล้วก็ รอ

ใครอยากเขียน sniff เองได้นะ ตามนี้เลย [Wireless "Deauth" Attack using Aireplay-ng, Python, and Scapy](http://raidersec.blogspot.com/2013/01/wireless-deauth-attack-using-aireplay.html)

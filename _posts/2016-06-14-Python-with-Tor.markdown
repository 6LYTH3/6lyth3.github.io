---
layout: post
title:  "How to connect Tor with Python"
date:   2016-06-14 14:00:00 +0700
categories: Hacking
tags:
- Python
- Tor
- Hacking
---
เรารู้กันดีว่า Tor เป็น Anonymous proxy ที่ระบุตัวตนไม่ได้ แต่เราก็ต้องแลกมากกับความว่า `ช้า` ในเมื่อเรารู้กันดีอยู่แล้ว เรามาเริ่มใช้งานกันเลยดีกว่า ผมแนะนำให้ Install ผ่าน Package management จะดีกว่า Download browser มาลง

For MacOS
{% highlight shell %}
# Install via HomeBrew
$ brew install tor

# Start Tor
$ tor
{% endhighlight %}

หลังจากลงเสร็จจะมี configuration file ที่เราควรจะเอาไปเพื่ออะไรนิดหน่อย สำหรับ MacOS file จะอยู่ที่ `/usr/local/etc/tor/torrc` ให้เพิ่มไปตามนี้
{% highlight shell %}
$ cat /usr/local/etc/tor/torrc
# For new Identity
ControlPort 9051
{% endhighlight %}

เรามาเริ่มเขียน Python ต่อไปยัง Tor กัน แต่ก่อนอื่นต้องลง Lib `socks` `requests` or `urllib2` หรืออะไรก็ได้ที่ส่ง http request ได้
{% highlight python %}
#!/usr/bin/env python
"""
Author: Blythe LilYoojun
Lib Require: socks and Tor proxy
"""
import socket
import socks
import requests

ipcheck_url = 'http://checkip.amazonaws.com/'

def connentTor():
	socks.setdefaultproxy(socks.PROXY_TYPE_SOCKS5, '127.0.0.1', 9050)
	socket.socket = socks.socksocket

if __name__ == '__main__':
  connentTor()
  print(requests.get(ipcheck_url).text)
{% endhighlight %}

ผลลัพธ์
{% highlight shell %}
$ pyhton simpleTor.py
5.214.112.76
{% endhighlight %}

เท่านี้ไม่พอ เรายังอยากได้ เปลี่ยน IP Address ทุกครั้งที่มีการส่ง request
ทุกครั้งที่เรา start `tor` มันจะเปิด port ตามที่ config ไว้ในไฟล์ `torrc` ไว้ให้เราเข้าไป control อะไรบ้างอย่าง แต่ในที่นี้เราจะส่งสัญญาณบอก tor ว่าเราจะของ IP ใหม่นะ ซึ่งเราสามารถลองส่งค่าต่างๆ ด้วยการ telnet ไปยัง port 9051 ได้เลย
{% highlight shell %}
$ telnet localhost 9051
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
AUTHENTICATE
250 OK
SIGNAL NEWNYM
250 OK
{% endhighlight %}

อีก Shell หนึ่งเปิด run simpleTor.py จะได้ ip ใหม่มา ทุกๆ การส่งสัญญาณ NEWNYM
{% highlight shell %}
$ pyhton simpleTor.py
17.100.43.8
{% endhighlight %}

ได้แล้ว เรามาลองเขียนเป็น program กันดู
{% highlight python %}
#!/usr/bin/env python
"""
Author: Blythe LilYoojun
Lib Require: socks and Tor proxy
"""
import socket
import socks
import requests

ipcheck_url = 'http://checkip.amazonaws.com/'

def connentTor():
	socks.setdefaultproxy(socks.PROXY_TYPE_SOCKS5, '127.0.0.1', 9050)
	socket.socket = socks.socksocket

def getNewIP():
  socks.setdefaultproxy()
  s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
  s.connect(("127.0.0.1", 9051))
  s.send("AUTHENTICATE\r\n")
  r = s.recv(128)
  if r.startswith("250"):
  	s.send("SIGNAL NEWNYM\r\n")
	s.close()
  connentTor()

if __name__ == '__main__':
  connentTor()
  print(requests.get(ipcheck_url).text)

  getNewIP()
  print(requests.get(ipcheck_url).text)
{% endhighlight %}

เท่านี้เราก็จะได้ Python script ที่ต่อไปยัง Tor แล้วได้ IP ใหม่ทุกๆ การ request แล้วครับ


Reference

[Tor New Identity](https://stem.torproject.org/faq.html)

[Simple Config for MacOS](https://kremalicious.com/simple-tor-setup-on-mac-os-x/)

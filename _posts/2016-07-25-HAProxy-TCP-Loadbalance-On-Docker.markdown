---
layout: post
title:  "HAProxy TCP Load balance on Docker"
date:   2016-07-25 13:10:00 +0700
categories: DevOps
tags:
 - Docker
 - HAProxy
---
มีโจทย์ว่าจะต้องทำ Load balance มาครอบ Middleware ชุดหนึ่ง ก็เลยจะเอา `HAProxy` มาใช้ แต่เคยใช้แต่ใน mode http ก็เลยไม่มั่นใจว่าจะทำได้ไหม เลยลองทำ Lab เล่นดู

มาเริ่มกันเลย ผมจะทำ TCP Server กับ Client ขึ้นมาก่อน

Simple TCP Server.go
{% highlight go %}
package main

import (
  "fmt"
  "log"
  "net"
)

func main() {
  fmt.Println("Hey Baby, I glad to see you again")
  ln, err := net.Listen("tcp", ":9000")
  if err != nil {
    log.Fatal(err)
  }
  defer ln.Close()

  for {
    // Wait for connection
    conn, err := ln.Accept()
    if err != nil {
      log.Fatal(err)
    }   

    go handleConnection(conn)
  }
}

func handleConnection(conn net.Conn) {
  // Send Hello
  conn.Write([]byte("Send by Worker 1"))
  conn.Close()
}
{% endhighlight %}

มาดูฝั่ง Client.go กันบ้าง

{% highlight go %}
package main

import (
  "fmt"
  "log"
  "net"
)

func main() {
  fmt.Println("Hi Honey")
  conn, err := net.Dial("tcp", "192.168.99.100:9900") // Server IP
  if err != nil {
    log.Fatal(err)
  }
  defer conn.Close()

  buf := make([]byte, 1024)
  bl, err := conn.Read(buf)
  if err != nil || bl == 0 {
    log.Fatal(err)
  }
  fmt.Println(string(buf))
}
{% endhighlight %}

ทดลอง run ดูก่อนว่าทำงานได้ไหม
{% highlight bash %}
(server) $ go run server.go      
Hey Baby, I glad to see you again
{% endhighlight %}

{% highlight bash %}
(client) $ go run client.go      
Hi Honey
Send by Worker 1
{% endhighlight %}

ทุกอย่างเป็นไปได้สวย ต่อไปเราลองมาจับ server.go ยัดลงไปใน Docker เพื่อทำให้เหมือนสถานการณ์จริง ผมเปลี่ยนเป็น worker1 กับ worker2 เพื่อที่จะได้ระบุถูกว่า client เราต่อไปที่เครื่องไหน

Create worker1 (สร้าง directory worker1 แล้วเรา server.go ไปว่างรอไว้)
{% highlight bash %}
$ docker run -d --name worker1 -p 9000:9000 -v `pwd`/worker1:/go golang:alpine go run server.go
{% endhighlight %}

Create worker2
{% highlight bash %}
docker run -d --name worker2 -p 9001:9001 -v `pwd`/worker2:/go golang:alpine go run server.go
{% endhighlight %}

ต่อมาสร้าง file haproxy.cfg แบบง่ายๆ ก่อนกัน
{% highlight bash %}
global
  maxconn 45000
  daemon

defaults
  timeout server 10s
  timeout client 30s
  timeout connect 30s

listen foreman
  bind *:9900
  mode tcp
  balance roundrobin
  server w1 worker1:9000
  server w2 worker2:9001
{% endhighlight %}
เราใช้ mode เป็น tcp และ algorithm เป็น roundrobin คือสลับไปเรื่อยๆ

Create haproxy1
{% highlight bash %}
docker run -d --name haproxy1 -p 9900:9900 --link worker1:worker1 --link worker2:worker2 -v `pwd`/haproxy/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro haproxy:alpine
{% endhighlight %}
link ที่กำหนดลงไปมาจาก Container name ของ TCP Server ที่ทำไว้ก่อนหน้า

*จริงๆ แล้วเราไม่ต้อง expose port ของ worker ก็ได้ เพราะ link ใน docker มันจะทำการ map port ให้เราเอง*


List process ดูว่าทุกตัว run อยู่
{% highlight bash %}
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
585f3c43ffef        haproxy:alpine      "/docker-entrypoint.s"   24 seconds ago      Up 23 seconds       0.0.0.0:9900->9900/tcp   haproxy1
cb134a0672b3        golang:alpine       "go run server.go"       2 minutes ago       Up 2 minutes        0.0.0.0:9001->9001/tcp   worker2
3c71a5d835ba        golang:alpine       "go run server.go"       3 minutes ago       Up 3 minutes        0.0.0.0:9000->9000/tcp   worker1
{% endhighlight %}

ลองมาเทสกันครับ
{% highlight bash %}
$ go run client.go
Hi Honey
Send by Worker 1
$ go run client.go
Hi Honey
Send by Worker 2
$ go run client.go
Hi Honey
Send by Worker 1
$ go run client.go
Hi Honey
Send by Worker 2
{% endhighlight %}

Good jobs, well done!


[Haproxy configuration](http://www.haproxy.org/download/1.4/doc/configuration.txt)

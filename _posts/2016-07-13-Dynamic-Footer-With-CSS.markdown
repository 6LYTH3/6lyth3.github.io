---
layout: post
title:  "Easy Way To Set Dynamic Footer with CSS"
date:   2016-07-13 15:00:00 +0700
categories: DevOps
tags: CSS
---
หลายวันก่อนผมพยายาม `Footer` ของผมเปลี่ยนไปตาม content ของ page ซึ่งบาง page โชว์ content ไม่ถึง 100% ของ page บาง page ก็ยาวเกิน ทำให้ Footer ทำงานไม่ถูกต้อง

วิธีนี้เป็นวิธีง่ายๆ ในการ set ให้ Footer เปลี่ยนไปตาม content ของ page

{% highlight css %}
html, body {
  height: 100%;
  margin: 0;
}

main {
  min-height: 96%;
}

main:after {
  content: "";
  display: block;
}

main:after, footer {
  height: 10px;
}
{% endhighlight %}

ในส่วนของ HTML Page เราก็เขียนประมาณนี้ สร้าง tag สำหรับ main และ footer ครอบเอาไว้ก่อน

{% highlight html %}
<body>
<!-- Here is Main section -->
<main>
<div class="container">
  <div class="page-header">
    <h1>BLYTHE <small>I AM</small></h1>
  </div>
</div>
</main>

<!-- Footer present here -->
<footer>
<div class="container">
  Make with Love BLYTHE LilYoojun
</div>
</footer>

</body>
{% endhighlight %}

![Footer](/images/Footerx.png)

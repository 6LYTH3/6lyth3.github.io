---
layout: post
title:  "สร้าง Single Page สำหรับ Theme ของเรา"
date:   2016-06-08 13:00:00 +0700
categories: DevOps
tags:
- WordPress
---
ก่อนหน้านี้เราใช้ `index.php` สำหรับแสดงทุกๆ post เลย แต่ในความเป็นจริงเราความจะแยก แต่ละ post ออกเป็นอีก page หนึ่งด้วย เพื่อที่จะงานต่อการจัดการ ซึ่งส่วนนี้เราเรียกว่า single page

จริงๆ แล้ว WordPress มี Engine ที่ค่อยข้างฉลาดเอามากๆ ในการ เลือกดูว่า page นี้ควรจะเอาอะไรมาแสดง สามารถดูได้จาก WordPress hierarchy ข้างล่างเลย

`single.php` พื้นฐานแล้วจะเหมือนกับ index เกือบทุกอย่าง จะต่างตรง layout หรือ style ก็เพียงเท่านั้น

{% highlight php %}
<?php get_header(); ?>
<?php
if ( have_posts() ) {
  while ( have_posts() ) {
    the_post();
    ?>  
    <h1><? echo the_title(); ?></h1>
    <? echo the_content(); ?>

    <?  
  } // end while
} // end if
?>


<?php comments_template(); ?>  
<?php get_footer(); ?>
{% endhighlight %}

ตรงนี้ผมได้เพื่อ `comments_template` เข้ามาในหน้า single ด้วย เพื่อที่จะได้ มีก้อง comment ให้ได้ใช้งาน

![WordPress hierarchy](http://marktimemedia.com/wp-content/uploads/2013/05/template-hierarchy-desktop-dark-2015.png)

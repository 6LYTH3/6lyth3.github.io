---
layout: post
title:  "สร้าง WordPress Theme ของเรากัน"
date:   2016-06-07 18:00:00 +0700
categories: DevOps
tags:
- WordPress
---
หลังจากที่เรา Install WordPress กันไปแล้ว คราวนี้มาลองสร้าง Theme ของเรากัน ซึ่งจริงๆ แล้ว ส่วนที่เล็กที่สุด สำหรับใช้ทำ Theme ใน WordPress คือ `index.php` กับ `style.css` แค่ 2 file นี้ เราก็สามารถสร้าง Theme ของเราเองได้แล้ว

เริ่มจากเข้าไปที่ wp-content แล้วสร้าง folder สำหรับ Theme ของเราขึ้นมาก่อน

{% highlight shell %}
$ cd wp-content/themes
$ mkdir mythemes && cd !$
$ touch index.php style.css
{% endhighlight %}

ก่อนที่เราจะ Active theme ของเราในหน้า Admin เราต้องทำการใส่ information บ้างอย่างให้กับ style.css ของเราก่อน โดยสามารถดูรายละเอียดต่างๆ ได้ที่ [Theme Stylesheet](https://codex.wordpress.org/Theme_Development)

ในที่นี้ผมใส่ประมาณนี้ `style.css`

{% highlight css %}
/*
Theme Name: MYThemes
Theme URI: http://6lyth3.github.io/
Author: Blythe LilYoojun
Author URI: http://6lyth3.github.io/
Description: The 2016 theme for New era of WordPress.
Version: 1.0
*
{% endhighlight %}

จากนั้นก็ Active theme ให้งานได้ แต่มันจะเป็นว่างๆ ใช่ไหม ฮ่าๆ

ส่วนสำคัญต่อมาในการทำ Home page ของ WordPress เลยคือ [The Loop](https://codex.wordpress.org/The_Loop) คือส่วนที่ WordPress ใช้ในการ Query หา post ที่จะมาโชว์ใน page ที่เราต้องการ

เราลองให้ The Loop ดึง post ทั้งหมดทีมีมาโชว์ในหน้า index.php ดู ด้วยการเพิ่ม code ลงไปนิดหน่อยก่อน

{% highlight php %}
<?php
if (have_posts()) {
  while (have_posts()) {
    the_post();
    ?>  
    <a href="<?php the_permalink(); ?>"><h1><? echo the_title(); ?></h1></a>
    <? echo the_excerpt(); ?>
    <?  
    } // end while
  } // end if
?>
{% endhighlight %}

เท่านี้เราก็จะได้ Post ทั้งหมดออกมาแสดงแล้ว

นี้เป็นเพียงการเริ่มต้นสร้าง Theme แบบง่ายๆ จาก 2 ไฟล์พื้นฐานที่สำคัญของ WordPress

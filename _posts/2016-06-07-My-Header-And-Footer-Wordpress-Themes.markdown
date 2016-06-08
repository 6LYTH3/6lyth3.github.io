---
layout: post
title:  "สร้าง Header และ Footer สำหรับ Theme ของเรา"
date:   2016-06-07 19:00:00 +0700
categories: DevOps
tags:
- WordPress
---
ครั้งนี้เรามาสร้าง Header และ Footer กัน ซึ่งใน WordPress ก็จะมี name conversation สำหรับสร้างสิ่งเหล่านี้อยู่ สำหรับ Header ก็คือ `header.php` และ Footer ก็คือ `footer.php`

มาทำไฟล์ `header.php` ง่ายๆ กันก่อน

{% highlight php %}
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title><?php bloginfo('name'); ?> | <?php is_home() ? bloginfo('description') : wp_title( '' ); ?></title>
  <link rel="stylesheet" href="<?php echo get_stylesheet_uri(); ?>" media="screen" title="no title" charset="utf-8">
</head>
<body <?php body_class($class); ?>>
  <h1>BLYTHE</h1>
{% endhighlight %}

ในนี้ จุดที่น่าสนใจคือ `<title>` กับ `body_class` ซึ่งตรง title เราอยากจะให้มัน title จริงๆ ของ post นั้นๆ ปัญหามันอยู่ตรงที่หน้า `index.php` มันไม่มี title เราเลยไปเอา blog description มาโชว์แทน

ส่วน body_class มีประโยชน์มากๆ คือ WordPress มันจะเพิ่ม class แต่ๆ ที่เป็นไปตาม class hierarchy ของมัน เช่นถ้าเราอยู่ในหน้า single post มันก็จะเพิ่ม class single มาให้เราอัตโนมัติ

ต่อมาเป็น `footer.php`

{% highlight html %}
Copyright 2016
</body>
</html>
{% endhighlight %}

โคตร classic เลยสำหรับ Footer

เวลาจะเรียกใช้งาน Header Footer ก็ไม่ยาก เพิ่งแค่ใส่ code ตามนี้

Header เพิ่มในส่วนบนสุดของ file
{% highlight php %}
<?php get_header(); ?>
{% endhighlight %}

Footer เพิ่มในส่วนล่างสุดของ file
{% highlight php %}
<?php get_footer(); ?>
{% endhighlight %}

---
layout: post
title:  "สร้าง Function สำหรับ Register Default Function ต่างมายัง Theme ของเรา"
date:   2016-06-08 13:20:00 +0700
categories: DevOps
tags:
- WordPress
---
ง่ายๆ เลยคือ สร้าง file `function.php` ขึ้นมาใน Theme ของเรา function ต่างๆ ที่เราได้เขียนไว้ในไฟล์นี้ ถ้าเรานำไปใช้ใน file อื่นๆ ภายใต้ Theme ของเรา WordPress จะรู้โดยอัตโนมัติว่าจะต้องไปใช้ function ใน file function.php นะ

{% highlight php %}
<?php
  /* WP function */
  add_theme_support( 'post-thumbnails' );
  register_sidebar();

  function aboutMe()
  {
    echo "I'm Blythe LilYoojun";
  }

 ?>
?>

ในที่นี้ผมมี function ง่ายๆ คือ `aboutMe` ใช้แสดงชื้อผมเท่านั้น เวลาเรียกใช้งานจากที่อ่ื่นก็เพียงเพิ่ม code  `<? aboutMe(); ?>` ลงไปใน php ของเรา

อีก 2 อันที่น่าสนใจคือ `add_theme_support` กับ `register_sidebar` คือการเพิ่มรูป thumbnails ของ post นั้นๆ นั้นเอง และอีกอันก็คือการเพิ่ม sidebar

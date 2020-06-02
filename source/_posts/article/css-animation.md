---
title: CSS3新特性之animation
date: 2020-06-02 18:02:48
summary: CSS3新特性之animation (纯CSS实现轮播图)
categories: 文章
tags: 
- CSS 3
- animation
top:
cover:
img:
keywords:
---

## HTML
```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <link rel="stylesheet" href="./css/index.css">
    </head>
    <body >

        <div class="swiper-wrap">
            <ul class="swiper-img">
                这里的<!---->是为了消除每个li换行所带来的间距问题
                <li><img src="./images/1.jpg" alt="" /></li><!--
                --><li><img src="./images/2.jpg" alt="" /></li><!--
                --><li><img src="./images/3.jpg" alt="" /></li><!--
                --><li><img src="./images/4.jpg" alt="" /></li><!--
                --><li><img src="./images/5.jpg" alt="" /></li><!--
                --><li><img src="./images/6.jpg" alt="" /></li>
            </ul>
           

        </div> 
        
        <script src="./js/index.js"></script>
    </body>
</html>
```

## CSS
```css
body{
    padding: 0;
    margin: 0;
}
ul{
    margin: 0;
    padding: 0;
}
.swiper-wrap{
    width: 100%;
    position: absolute;
    top: 0;
    bottom: 0;
    overflow: hidden;
}
.swiper-wrap li{
    display: inline-block;
}
.swiper-wrap .swiper-img{
    width: 600%;
    height: 100%;
    border: 1px solid darkcyan;
}
.swiper-wrap .swiper-img li{
    width: calc(100%/6);
    height: 100%;
}
.swiper-wrap .swiper-img li img{
    width: 100%;
    height: 100%;
}

/* 切换 */
.swiper-img{
    animation: swiper 12s linear infinite;
}
@keyframes swiper{
    0% { margin-left: 0%;}
    9.1% { margin-left: 0%;}
    18.2% { margin-left: -100%;}
    27.3% { margin-left: -100%;}
    36.4% { margin-left: -200%;}
    45.5% { margin-left: -200%;}
    54.6% { margin-left: -300%;}
    63.7% { margin-left: -300%;}
    72.8% { margin-left: -400%;}
    81.9% { margin-left: -400%;}
    91% { margin-left: -500%;}
    100% { margin-left: -500%;}

}
```

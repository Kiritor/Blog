title: canvas画图集锦
date: 2016-04-6 15:30:00
tags: [canvas]
categories: front
---
最近有点闲了,学习了下H5的canvas,用canvas来画图感觉挺有意思的,创意是最关键的(榆木脑袋,没啥创意).学习中通过canvas随意画了几幅案例图,想着没事记录下来也挺好的.
## 表情系列
通过canvas绘制一些表情,不定期更新......
<!--more-->
### 笑脸1
先画个哈哈大笑的笑脸压压惊,效果图:
<center>![笑脸](/img/smile1.png)</center>
代码如下:
```html
<html>

<head>
    <title>笑脸</title>
    <style>
        body {
            text-align: center;
        }
    </style>
</head>
<bdoy>
    <canvas id="can1" width="800px" height="800px"></canvas>
    <script>
        function draw(id) {
            var can1 = document.getElementById(id);
            if (can1 == null)
                return false;
            var context = can1.getContext("2d");
            //画脸
            context.fillStyle = "#58c7c7";
            context.beginPath();
            context.arc(400, 400, 140, 0, 2 * Math.PI, false);
            context.fill();
            context.closePath();
            //画眼睛
            context.fillStyle = "#ffffff";
            context.beginPath();
            context.arc(360, 350, 20, 0, 2 * Math.PI, false)
            context.fill();
            context.closePath();
            context.beginPath();
            context.arc(440, 350, 20, 0, 2 * Math.PI, false)
            context.fill();
            context.closePath();
            //画嘴巴
            context.fillStyle = '#ffffff';
            context.beginPath();
            context.arc(400, 420, 80, 0, -1 * Math.PI);
            context.fill();
            context.closePath();
            
        }
        draw("can1");
    </script>
</bdoy>

</html>
```
### 笑脸2
接下来,来个腼腆的,效果图如下:
<center>![浅笑](/img/smile2.png)</center>
只需要在上面的基础上,稍微的修改下代码即可:
```html
<html>

<head>
    <title>笑脸</title>
    <style>
        body {
            text-align: center;
        }
    </style>
</head>
<bdoy>
    <canvas id="can1" width="800px" height="800px"></canvas>
    <script>
        function draw(id) {
            var can1 = document.getElementById(id);
            if (can1 == null)
                return false;
            var context = can1.getContext("2d");
            //画脸
            context.fillStyle = "#58c7c7";
            context.beginPath();
            context.arc(400, 400, 140, 0, 2 * Math.PI, false);
            context.fill();
            context.closePath();
            //画眼睛
            context.fillStyle = "#ffffff";
            context.beginPath();
            context.arc(360, 350, 20, 0, 2 * Math.PI, false)
            context.fill();
            context.closePath();
            context.beginPath();
            context.arc(440, 350, 20, 0, 2 * Math.PI, false)
            context.fill();
            context.closePath();
            //画嘴巴
            context.lineWidth = "4"; //线框宽度
            context.strokeStyle ="#ffffff";
            context.beginPath();
            context.arc(400, 400, 80, Math.PI/6,  Math.PI*5/6);
            context.stroke();;
            context.closePath();
            
        }
        draw("can1");
    </script>
</bdoy>

</html>
```
### 难过
看着这个表情，好像就难过起来了呢.
<center>![难过](/img/sad.png)</center>
也只是调整下嘴巴的画法就可以了:
```html
<html>

<head>
    <title>笑脸</title>
    <style>
        body {
            text-align: center;
        }
    </style>
</head>
<bdoy>
    <canvas id="can1" width="800px" height="800px"></canvas>
    <script>
        function draw(id) {
            var can1 = document.getElementById(id);
            if (can1 == null)
                return false;
            var context = can1.getContext("2d");
            //画脸
            context.fillStyle = "#58c7c7";
            context.beginPath();
            context.arc(400, 400, 140, 0, 2 * Math.PI, false);
            context.fill();
            context.closePath();
            //画眼睛
            context.fillStyle = "#ffffff";
            context.beginPath();
            context.arc(360, 350, 20, 0, 2 * Math.PI, false)
            context.fill();
            context.closePath();
            context.beginPath();
            context.arc(440, 350, 20, 0, 2 * Math.PI, false)
            context.fill();
            context.closePath();
            //画嘴巴
            context.lineWidth = "4";
            context.strokeStyle ="#ffffff";
            context.beginPath();
            context.arc(400, 510, 80, Math.PI+Math.PI/6,Math.PI*2-Math.PI/6);
            context.stroke();;
            context.closePath();
            
        }
        draw("can1");
    </script>
</bdoy>

</html>
```
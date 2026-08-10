# css常用手写功能
# <font color="#e96900">自适应画一个正方形</font>

## 1、vw
vh and vw：相对于视口的高度和宽度，而不是父元素的（CSS百分比是相对于包含它的最近的父元素的高度和宽度）。1vh 等于1/100的视口高度，1vw 等于1/100的视口宽度。
```html
<style>
/* 第二种vw vh 相对于viewport视窗的宽高进行计算的单位  兼容性ie9*/
#box{
  width: 50vw;
  /* 移动端适配 因为是长方形屏幕 height需要设置为vw 才能是正方形 */
  height: 50vw;
  background: lightblue;
}
</style>

  <div id="box">asd</div>

```
注意！！！ height不能设置成`vh`，移动端是长方形的 `20vw != 20vh`，宽高都设置成`vw`就好了

---

## 2、padding
在 CSS 盒模型中，一个比较容易被忽略的就是 `margin, padding` 的百分比数值计算。按照规定`margin, padding` 的百分比数值是`相对 父元素 的宽度`计算的。由此可以发现只需将元素垂直方向的一个 `padding` 值设定为与 `width` 相同的百分比就可以制作出自适应正方形了
```html
<style>
  #box{
    /* 横向50% */
    width: 50%; 
    /* 纵向50% padding百分比是相对父元素的宽度 */
    padding-bottom:50%;
    /* height设为0 避免有内容撑开盒子大小 */
    height: 0;
    background-color: lightblue; 
  }
</style>

 <div id="box">asd</div>
```
设置`padding-bottom`内容会正常显示，设置成`padding-top`内容会显示在文字的下方

但要注意，仅仅设置`padding-bottom`是不够的，若向容器添加内容，内容会占据一定高度，为了解决这个问题，需要设置height: 0。

优点：简洁明了，且兼容性好。

缺点：会导致在元素上设置的max-width属性失效（max-height不收缩）。

---

## 3、伪类的margin/padding-top 撑开容器

`伪类都是相对父元素的宽高的` 本身`margin/padding`的百分比就是`相对父元素的宽度计算的`， 所以伪类设置`100%` 就是跟父元素的宽度是一样的 都是相对于页面`50% `就是一个正方形了

`这里需要注意的是元素有无内容的区分`

```html
<style>
/* 无内容时 padding实现*/
#box{
width:50%;
background: lightblue;
}
#box::after{
content:'';
display: block;
padding-top:100%;
}

/* ########## */

/* 无内容时 margin实现 因为在垂直方向上盒子和伪类发生了外边距重叠触发BFC （padding不会触发） 所以父元素上添加 overflow: hidden;*/
#box{
width:50%;
background: lightblue;
overflow: hidden;
}
#box::after{
content:'';
display: block;
margin-top:100%;
}
</style>
<div id="box"></div>
```
```html
<style>
/* 
有内容时 无论是使用margin还是padding当有内容时都会出现高度溢出的问题 高度发生变化 不再是一个正方形
可以利用绝对定位消除空间占用 并且布局的时候内容单独放在一个div中
*/

/* 有内容时 padding */
.container{
width:50%;
background: lightblue;
position: relative;
}
.container::after{
content: "";
display: block;
padding-top: 100%;
} 
/* ::after 是在内容后添加内容 没有使用绝对定位的时候 宽高也算在盒子的大小 所以height：100% 是占到了padding的大小*/
.container .box{
position: absolute;
width: 100%;
height: 100%; 
} 


/* ########## */

/* 有内容时 margin */
.container{
width:50%;
background: lightblue;
position: relative;
overflow: hidden;
}
.container::after{
content: "";
display: block;
margin-top: 100%;
}
.container .box{
position: absolute;
width: 100%;
height: 100%; 
}
</style>

 <div class="container"> 
    <div class="box">有内容</div>
  </div>
```
 --- 

## 4、 rem
动态设置根元素的`font-size`，然后利用`rem`对元素的`width、height`进行布局
动态设置： 1、`媒体查询` 根据不同屏幕大小动态配置根元素的`font-size`；2、`js`监听窗口变化动态改变根元素的`font-size`

---

# <font color="#e96900">实现一个三角形</font>
```css
.sjx {
  width: 0;
  height: 0;
  margin: 100px auto;
  border-top: 50px solid transparent;
  border-left: 50px solid transparent;
  border-right: 50px solid transparent;
  border-bottom: 50px solid red;
}
```
# <font color="#e96900">实现一个下拉箭头</font>
```css
.jt{
  width: 50px;
  height: 50px;
  border: 5px solid transparent;
  border-left:5px solid red;
  border-top:5px solid red;
  transform: rotate(-135deg);
}
```
# <font color="#e96900">实现一个圆形</font>
```css
.circle1 {
  width: 10vw;
  height: 10vw;
  background: lightblue;
  border-radius: 50%;
}
.circle2 {
  width: 10vw;
  height: 10vw;
  background: lightcoral;
  clip-path: circle();
}

.circle3 {
  width: 10vw;
  height: 10vw;
  background: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg'%3E%3Ccircle cx='50%25' cy='50%25' r='50%25' fill='gray'/%3E%3C/svg%3E");
}

.circle4 {
  width: 10vw;
  height: 10vw;
  background: radial-gradient(gray 70%, transparent 70%);
}

.circle5::after {
  content: "●";
  font-size: 10vw;
  line-height: 1;
  color: antiquewhite;
}
```
# <font color="#e96900">实现一个五角星</font>
```css
#star-five {
  margin: 50px 0;
  position: absolute;
  display: block;
  color: red;
  width: 0;
  height: 0;
  border-right: 100px solid transparent;
  border-bottom: 70px solid red;
  border-left: 100px solid transparent;
  transform: rotate(35deg);
  left: 200px;
}
#star-five:before {
  border-bottom: 80px solid red;
  border-left: 30px solid transparent;
  border-right: 30px solid transparent;
  position: absolute;
  height: 0;
  width: 0;
  top: -45px;
  left: -65px;
  display: block;
  content: '';
  transform: rotate(-35deg);
}
#star-five:after {
  position: absolute;
  display: block;
  color: red;
  top: 3px;
  left: -105px;
  width: 0px;
  height: 0px;
  border-right: 100px solid transparent;
  border-bottom: 70px solid red;
  border-left: 100px solid transparent;
  transform: rotate(-70deg);
  content: '';
}
```
实现思路：三个三角形旋转叠加
# <font color="#e96900">手写一个满屏品字布局方案</font>
```html
<style>
  html,
  body {
    height: 100%;
    margin: 0;
    padding: 0;
  }

  .container {
    display: flex;
    flex-direction: column;
    flex-wrap: wrap;
    width: 100%;
    height: 100%;
  }

  .firstRow,
  .secondRow {
    width: 100%;
    height: 30%;
    display: flex;
    flex-direction: row;
    justify-content: center;
    margin: 10px 0;
  }

  .item {
    background-color: red;
    width: 40%;
    height: 100%;
    margin: 0 auto;
    border-radius: 10%;
  }
</style>

<article class="container">
  <div class="firstRow">
    <div class="item"></div>
  </div>
  <div class="secondRow">
    <div class="item"></div>
    <div class="item"></div>
  </div>
</article>
```
# <font color="#e96900">实现一个圆环进度条</font>
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        .con {
            position: relative;
            display: inline-block;
            height: 200px;
            width: 200px;
        }

        .percent-circle {
            position: absolute;
            height: 100%;
            background: #f00;
            overflow: hidden;
        }

        .percent-circle-right {
            right: 0;
            width: 100px;
            border-radius: 0 100px 100px 0/0 100px 100px 0;
        }

        .percent-circle-right .right-content {
            position: absolute;
            content: '';
            width: 100%;
            height: 100%;
            transform-origin: left center;
            transform: rotate(0deg);
            border-radius: 0 100px 100px 0/0 100px 100px 0;
            background: #bbb;
        }

        .percent-circle-left {
            width: 100px;
            border-radius: 100px 0 0 100px/100px 0 0 100px;
        }

        .percent-circle-left .left-content {
            position: absolute;
            content: '';
            width: 100%;
            height: 100%;
            transform-origin: right center;
            transform: rotate(0deg);
            border-radius: 100px 0 0 100px/100px 0 0 100px;
            background: #bbb;
        }

        .text-circle {
            position: absolute;
            display: flex;
            align-items: center;
            justify-content: center;
            height: 80%;
            width: 80%;
            left: 10%;
            top: 10%;
            border-radius: 100%;
            background: #000;
            color: #fff;
        }
    </style>
</head>

<body>
    <div class="con">
        <div class="percent-circle percent-circle-left">
            <div class="left-content"></div>
        </div>
        <div class="percent-circle percent-circle-right">
            <div class="right-content"></div>
        </div>
        <div class="text-circle">0%</div>
    </div>
</body>
<script>
    var leftContent = document.querySelector(".left-content");
    var rightContent = document.querySelector(".right-content");
    var textCircle = document.querySelector(".text-circle");

    //先是leftContent旋转角度从0增加到180度，
    //然后是rightContent旋转角度从0增加到180度
    var angle = 0;

    var timerId = setInterval(function () {
        angle += 30;
        if (angle > 360) {
            clearInterval(timerId);
        } else {
            if (angle > 180) {
                rightContent.setAttribute('style', 'transform: rotate(' + (angle - 180) + 'deg)');
            } else {
                leftContent.setAttribute('style', 'transform: rotate(' + angle + 'deg)');
            }
            setPercent(angle);

        }
    }, 1500);

    function setPercent(angle) {
        textCircle.innerHTML = parseInt(angle * 100 / 360) + '%';
    }

</script>

</html>
```
# <font color="#e96900">用css实现一个硬币旋转的效果</font>
```css
<!-- 两种实现方式：1、animation+keyframes 2、transition： -->
  <style type="text/css">
    .around { 
      width: 200px;
      height: 200px;
      background: orange;
      /*圆形的话看不出效果，所以这里border-radius没有取50%*/
      border-radius: 40%;
      transform: rotate(0deg);
      animation: move 3s ease;
    }

    @Keyframes move {
      0% {
        transform: rotate(0deg);
      }
      50% {
        transform: rotate(360deg);
      }
      100% {
        transform: rotate(0deg);
      }
    }
  </style>

  <style type="text/css">
    .around {
      width: 200px;
      height: 200px;
      background: orange;
      /*圆形的话看不出效果，所以这里border-radius没有取50%*/
      border-radius: 40%;
      transform: rotate(0deg);
      transition: transform 3s linear;
    }

    .around:hover {
      transform: rotate(360deg);
    }
  </style>
```

# <font color="#e96900">如何强制（自动）中、英文换行与不换行</font>
```css
  word-break:break-all;只对英文起作用，以字母作为换行依据
  word-wrap:break-word; 只对英文起作用，以单词作为换行依据
  white-space:pre-wrap; 只对中文起作用，强制换行
  white-space:nowrap; 强制不换行，都起作用
```
---

# <font color="#e96900">文字超出隐藏显示省略号</font>
## 单行省略
```css
 white-space:nowrap; 
 overflow:hidden; 
 text-overflow:ellipsis;
 不换行，超出部分隐藏且以省略号形式出现（部分浏览器支持）
```
## 多行省略
```css
使文字数量不同在相同的地方显示，给盒子加固定高度
 
overflow：hidden;
display：-webkit-box; 将盒子转换为弹性盒子
-webkit-line-clamp：2; 设置显示多少行
text-overflow：ellipsis; 文本以省略号显示   
-webkit-box-orient：vertical; 文本显示方式，默认水平
```
---
# <font color="#e96900">画一条0.5px的线</font>
## 1、采用meta viewport的方式
```html
<meta name="viewport" content="width=device-width, initial-scale=0.5, minimum-scale=0.5, maximum-scale=0.5"/>
```
这样子就能缩放到原来的0.5倍，如果是1px那么就会变成0.5px

要记得viewport只针对于移动端，只在移动端上才能看到效果

## 2、采用transform: scale()的方式
transform: scale(0.5,0.5);
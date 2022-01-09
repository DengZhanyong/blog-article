# 用CSS也可以任意改变PNG图标颜色？



在开发中图标跟我们是形影不离的，在开始之前先聊一些题外话，**图标有什么作用？**

- 通用图标可以代替文字说明，能更简洁地表达某个设计的意图，让用户看到图标就知道它代表什么？有什么用途？

- LOGO类图标可以起到品牌宣传，同时因图标更醒目，占据更大的位置，用户更容易点击。

- 看图片更符合人的视觉需求，简洁的图片能降低人的信息焦虑。试想我们如果看到一个满满的文字的页面，是不是常常有信息恐惧，可能就会选择退出。

但是大多数情况下只有图标是不行的，还需要配上文字说明，如果用户看到一个图标，花很多时间都不能理解它的意思，那图标的存在也失去了它应有的意义。



### 开发问题

1. 是不是经常会遇到这样的场景，在导航栏的图标默认显示为一种浅色，当鼠标移到上面时图标变为深色，起到反馈用户的作用。

2. 还有一些场景下使用图标来标识状态，比如红色表示错误，黄色表示警告，绿色表示成功。

如果是使用 png 格式的情况下，我们会叫UI设计师提供给我们不同颜色的图标。



### 正式开始

现在我们就来解决上面这个麻烦的事情，使用 `css` 给 `PNG` 图标赋予任意颜色

![img](https://www.dengzhanyong.com/PHP/images/1615344607.png)

现在UI设计师给了我们一个表示状态的图标，现在我们要使用CSS来改变它的颜色

![img](https://www.dengzhanyong.com/PHP/images/1615345303.jpg)

这里使用的是CSS3滤镜 `filter` 中的 `drop-shadow` 实现的，这个滤镜可以给元素或图片的非透明区域添加投影。

可能会有小伙伴要问了，他与 `box-shadow` 有什么区别，直接看效果：

![img](https://www.dengzhanyong.com/PHP/images/1615345612.jpg)

通过效果可以很明显的看出区别，`box-shadow` 是对 `box` 进行投影，而`drop-shadow` 是对内容进行投影，只有有内容的地方才会产生投影，这更符合我们现实生活中的现象。

它的参数形式为：**drop-shadow(*h-shadow v-shadow blur spread color*)**，与 `boxshadow` 参数的唯一区别就是无法指定为内投影。



### 投影变色

有了上面的了解后，现在加大在水平方向的投影距离，不设置阴影大小和颜色：

```css
img {
    animation: move 3s linear forwards;
}
@keyframes move {
    0% {
        filter: drop-shadow(0 0 #000);
    }
    100% {
        filter: drop-shadow(150px 0 #000);
    }
}
```



![img](https://www.dengzhanyong.com/PHP/images/1615346857.gif)

**隐藏原图标**：将原图标水平向左移动自身的100%，图标外层的 div 设置 `overflow:hide`，这样投影影子就替换了原图标的位置：

```html
<div class="img-wrapper">
    <img class="img" src="./icon.png" alt="">
</div>
```

```css
.img-wrapper {
    display: inline-block;
    border: 1px solid #ccc;
}
.img {
    transform: translateX(-100%);
    filter: drop-shadow(120px 0 #000);
}
```

为了演示整个过程，我这里没有设置超出隐藏：

![img](https://www.dengzhanyong.com/PHP/images/1615347867.gif)

现在你可以将图标改成任意颜色啦，不过这种方式有个局限性，只能变成纯色图标

**关键知识点：**

- 还有一点要注意， `drop-shadow` 不能像 `box-shadow` 一样设置多层阴影，否则无任何效果。

![img](https://www.dengzhanyong.com/PHP/images/1615353830.jpg)



**给文字投影**

`drop-shadow` 不仅可以给图片投影，还可以给元素投影。

```css
color: red;
font-size: 40px;
filter: drop-shadow(10px 10px 3px rgb(7, 7, 7));
```

![img](https://www.dengzhanyong.com/PHP/images/1615354166.jpg)

设想一下，如果把文字颜色设置为透明色，那投影是否会消失？

```css
.text-wrapper {
    color: red;
    font-size: 40px;
    filter: drop-shadow(10px 10px 3px rgb(7, 7, 7));
    animation: changeColor 3s linear;
}
@keyframes changeColor {
    0% {
        color: red;
    }
    100% {
        color: transparent;
    }
}
```

答案是会消失，这与我们的现实相符，阳光是可以穿过透明的物体的，因此不会产生投影。

![img](https://www.dengzhanyong.com/PHP/images/1615354497.gif)

对 `png` 图标改色的用处还是挺实用的，那通过它还可不可以做出一些其他有意思的事情呢？

比如说做一个小游戏，看轮廓猜一猜。点 [这里](https://www.dengzhanyong.com/resources/cssFilter/%E7%8C%9C%E8%BD%AE%E5%BB%93.html) 在线体验

![img](https://www.dengzhanyong.com/PHP/images/1615356159.png)



### 使用多滤镜变色

是不是以为这样就结束了，上面的变色方案实现起来还是比较麻烦的，通过 css 的其他滤镜同样可以实现变色效果。

先从简单的开始，把一个图标变为黑色或白色：

![img](https://www.dengzhanyong.com/PHP/images/1615358214.jpg)

```css
.state1 {
    filter: brightness(0);
}
.state2 {
    background: #000;
    filter: brightness(100);
}
```

使用到的是 `brightness` 滤镜，给图片应用一种线性乘法，使其看起来更亮或更暗。如果值是0%，图像会全黑。值是100%，则图像会变亮。



**如何变成其他颜色**

CSS3 filter还有很多其他的滤镜，如 `hue-rotate(色相)` 、`contrast(对比度)`、`saturate(饱和度)` 等。

CSS3 允许我们对统一元素同时应用多种滤镜，因此我们使用上面的滤镜，通过调整不同的参数组合，总会得到我们想要的颜色。

当然这个计算过程和原理是比较复杂的，我也没去研究，张鑫旭老师做了一个[在线转换器](https://www.zhangxinxu.com/sp/filter.html) ，只需要输入原始色（原图标颜色）和目标色（你想要变成的颜色），会自动帮我们生成滤镜代码：

![img](https://www.dengzhanyong.com/PHP/images/1615359847.jpg)

来看下使用滤镜的变化过程：

![img](https://www.dengzhanyong.com/PHP/images/1615359564.gif)

```css
.png {
    animation: change 3s linear forwards;
}
@keyframes change {
    0% {
        filter: none;
    }
    100% {
        filter: invert(52%) sepia(83%) saturate(839%) hue-rotate(78deg) brightness(111%) contrast(133%);
    }
}
```

这个在线转换器并不能保证每次都可以100%转换成目标颜色，它会存在一定的误差，误差值在2以内就可以了，细微的差别肉眼基本上区分不出来的。

![img](https://www.dengzhanyong.com/PHP/images/1615359903.gif)



### 使用混合模式变色

使用 `ackground-blend-mode` 实现变色有两个要求：

- 图片内容部分必须是纯黑色的
- 其余部分必须是白色的，不能是透明色

```html
<img src="./state.jpg">
<span class="colorful"></span>
```

```css
.colorful  {
    display: inline-block;
    width: 120px; height: 120px;
    background-image: url(./state.jpg), linear-gradient(#f4615c, #f4615c);
    background-blend-mode: lighten;
    background-size: 100%;
}
```

效果如下：

![img](https://www.dengzhanyong.com/PHP/images/1615361265.jpg)

**原理**：

​	将图片和目标色作为背景，使用 `lighten(变量)` 混合模式，白色与任何颜色混合都为白色，黑色与任何目标色混合后都为目标色。

​	这里的目标是之所以要使用线性渐变  `linear-gradient(#f4615c, #f4615c)` 的方式表示，而不直接使用 `#f4615c` ,可以看下我之前的这篇文章[《**奇思妙想CSS之“百变背景”**》](https://www.dengzhanyong.com/read/248)，你将会知道答案。

**`background-blend-mode` 的支持性还是很好的（IE不是浏览器）**

![img](https://www.dengzhanyong.com/PHP/images/1615362466.jpg)



### 总结

​		现在大多数情况下都是使用 `svg` 格式的图标，处理变色很方便。如果你用的是 `png` 图标，当有变色需求时就可以利用上面的方法，用户自定义主题时，也可以将图标颜色的配置权交给用户，做的真正的自定义主题。

 

最后，可通过公众号【前端筱园】，回复“**图标变色**” 获取本期示例全部源码；
































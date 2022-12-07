# CSS样式

## CSS3单位

### px

px像素（Pixel）。相对长度单位。像素px是相对于显示器屏幕分辨率而言的。

特点

- IE无法调整那些使用px作为单位的字体大小；
- 国外的大部分网站能够调整的原因在于其使用了em或rem作为字体单位；
- Firefox能够调整px和em，rem，但是96%以上的中国网民使用IE浏览器(或内核)。

### em

em是相对长度单位。相对于当前对象内文本的字体尺寸。如当前对行内文本的字体尺寸未被人为设置，则相对于浏览器的默认字体尺寸。

特点

- em的值并不是固定的；
- em会继承父级元素的字体大小。

**注意：**任意浏览器的默认字体高都是16px。所有未经调整的浏览器都符合: 1em=16px。那么12px=0.75em,10px=0.625em。为了简化font-size的换算，需要在css中的body选择器中声明Font-size=62.5%，这就使em值变为 16px*62.5%=10px, 这样12px=1.2em, 10px=1em, 也就是说只需要将你的原来的px数值除以10，然后换上em作为单位就行了。

所以我们在写CSS的时候，需要注意两点：

- body选择器中声明Font-size=62.5%；
- 将你的原来的px数值除以10，然后换上em作为单位；
- 重新计算那些被放大的字体的em数值。避免字体大小的重复声明。

也就是避免1.2 * 1.2= 1.44的现象。比如说你在#content中声明了字体大小为1.2em，那么在声明p的字体大小时就只能是1em，而不是1.2em, 因为此em非彼em，它因继承#content的字体高而变为了1em=12px。

### rem

rem是CSS3新增的一个相对单位（root em，根em），这个单位引起了广泛关注。这个单位与em有什么区别呢？区别在于使用rem为元素设定字体大小时，仍然是相对大小，但相对的只是HTML根元素。这个单位可谓集相对大小和绝对大小的优点于一身，通过它既可以做到只修改根元素就成比例地调整所有字体大小，又可以避免字体大小逐层复合的连锁反应。目前，除了IE8及更早版本外，所有浏览器均已支持rem。对于不支持它的浏览器，应对方法也很简单，就是多写一个绝对单位的声明。这些浏览器会忽略用rem设定的字体大小。下面就是一个例子：

```
p {font-size:14px; font-size:.875rem;}
```

## 常用文本样式属性

### color属性

- color属性可设置文本内容的前景色
- color属性主要可以用英语单词、十六进制、rgb()、rgba()等表示法
- rgba()表示法，最后一个参数表示透明度，介于0到1之间，0表示纯透明，1表示纯实心

### font-size属性

- font-size属性用来设置字号，单位通常是px，其他还有em、rem单位
- 网页文字正文字号通常是16px,浏览器最小支持10px字号

### font-weight属性

- font-weight属性设置字体的粗细程度，通常就用normal和bold两个值
- normal 正常粗细，与400等值
- bold 加粗，与700等值

### font-style

- font-style属性设置字体的倾斜
- normal 取消倾斜，比如可以把天生倾斜的i、em等标签设置为不倾斜
- italic 设置为倾斜字体
- oblique 设置为倾斜字体（用常规字体模拟，不常用）

### text-decoration属性

- text-decoration属性用于设置文本的修饰线外观的（下划线、删除线）
- none 没有修饰线
- underline 下划线
- line-through 删除线

## 字体属性

### font-family属性

- font-family属性用于设置字体

  ```
  font-family: "微软雅黑"
  ```

- 字体可以是列表形式，一般英语字体放到前面，后面的字体是前面的字体的后备字体

  ```
  font-family: serif,"Times New Roman","微软雅黑"
  ```

## 段落和行相关属性

### text-overflow属性

| 值 | 描述       | 
| ---------- | ---------- | 
| clip     | 修剪文本。                          | 
| ellipsis | 显示省略符号来代表被修剪的文本。     | 
| string | 使用给定的字符串来代表被修剪的文本。 | 

### text-indent属性

- text-indent属性定义首行文本内容之前的缩进量，缩进两个字符`text-indent: 2em;`

### line-height

- line-height属性用于定义行高
- line-height属性单位可以是px为单位的数值
- line-height属性也可以是没有单位的数值，表示字号的倍数`line-height:1.5`
- line-height属性也可以是百分数，表示字号的倍数`line-height:150%`
- 设置行高=盒子高度，即可实现单行文本垂直居中
- 设置text-align: center，即可实现文本水平居中

### font合写属性

- font属性可以用来作为font-style，font-weight，font-size，line-height和font-family属性的合写

  ```
  font: 20px/1.5 Arial,"微软雅黑";
  
  font: italic bold 20px/1.5 Arial,"微软雅黑";
  ```

## 继承性

- 文本相关的属性普遍具有继承性，只需要给祖先标签设置，即可在后代所有标签中生效。
- 因为文字相关属性有继承性，所以通常会设置body标签的字号、颜色、行高等，这样就能当做整个页面的莫仍样式了。

### 就近原则

- 在继承的情况下，选择器权重计算失效，而是“就近原则”

## 盒模型

- 所有HTML标签都可以看做矩形盒子，由width、height、padding、border构成，称为“盒模型”

### width、height不是盒子总宽高

- 盒子的总宽度 = width + 左右padding+左右border 
- 盒子的总高度 = height + 上下padding + 上下border

### width属性

- width属性表示盒子内容的宽度
- width属性的单位通常是px，移动端开发也会涉及百分数、rem等单位
- 当块级元素（div、h系列、li等）没有设置width属性时，它将自动撑满，但这并不意味着width可以继承

### height属性

- height属性表示盒子内容的高度
- height属性的单位通常是px，移动端开发也会涉及百分数、rem等单位
- 盒子的height属性如果不设置，它将自动被其内容撑开，如果没有内容，则height默认是0

### padding属性

- padding是盒子的内边距，即盒子边框内壁到文字的距离
- 四个方向的padding，可以分别用小属性进行设置。padding-top、padding-right、padding-bottom、padding-left
- padding属性如果用四个数值以空格隔开进行设置，分别上、右、下、左的padding
- padding属性如果用三个数值以空格隔开进行设置，分别表示上、左右、下的padding
- padding属性如果用两个数值以空格隔开进行设置，分别表示上下、左右的padding

### margin属性

- margin是盒子的外边距，即盒子和其他盒子之间的距离
- margin也有四个方向。margin-top、margin-right、margin-left、margin-bottom
- 竖直方向的margin有塌陷现象：小的margin会塌陷到大的margin中，从而margin不叠加，只以大值为准
- 一些元素（比如body、ul、p等）都有默认的margin，在开始制作网页的时候，要将他们清除
- 将盒子左右两边的margin都设置为auto，盒子将水平居中
- 文本居中是text-align: center;和盒子水平居中是两个概念

### box-sizing属性

- 将盒子添加了`box-sizing:border-box;`之后，盒子的width、height数字就表示盒子实际占有宽高（不含margin）了，即padding、border变为内缩，不再外扩
- box-sizing属性大量应用于移动网页制作中，因为它结合百分比布局、弹性布局等非常好用，在PC页面开发中使用较少

### display属性

- 使用`display:block;`即可将元素转为块级元素
- 使用`display:inline;`即可将元素转为行内元素，将元素转为行内元素的应用不多见
- 使用`display:inline-block;`即可将元素转为行内块

### 元素的隐藏

- 使用display:none;可以将元素隐藏，元素将彻底放弃位置，如果没有写它的标签一样
- 使用visibility:hidden;可以也可以将元素隐藏，但是元素不放弃自己的位置

## 浮动

### 浮动用来实现并排

- 浮动的最本质功能：用来实现并排
- 浮动使用要点：要浮动，并排的盒子都要设置浮动
- 父盒子要有足够的宽度，否则子盒子会掉下去

### 浮动的顺序贴靠特性

- 子盒子会按顺序进行贴靠，如果没有足够空间，则会寻找再前一个兄弟元素

### 浮动的元素一定能设置宽高

- 浮动的元素不再区分块级元素、行内元素，已经脱离了标准文档流，一律能够设置宽度和高度，即使它是span或者a标签等

### 右浮动

- float:right; 即可设置右浮动

### 注意事项

- 垂直显示的盒子，不要设置浮动，只有并排显示的盒子才要设置浮动
- 大盒子带着小盒子跑，一个大盒子中，又是一个小天地，内部可以继续使用浮动

### BFC规范

- BFC（Box Formatting Context，块级格式化上下文）是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素，反之亦然

### 如何创建BFC

- 方法一 float的值不是none
- 方法二 position的值不是static或者relative
- 方法三 display的值是inline-block、flex或者inline-flex
- 方法四 overflow:hidden

### 什么是overflow:hidden

- overflow:hidden; 表示溢出隐藏，溢出盒子边框的内容将会被隐藏
- overflow:hidden;是非常好用的让盒子形成BFC的方法

### BFC的其他作用

- BFC可以取消盒子的margin塌陷
- BFC可以阻止元素被浮动元素覆盖

### 清除浮动

- 清除浮动：浮动一定要封闭到一个盒子中，否则就会对页面后续元素产生影响

清除浮动方法一：让内部有浮动的父盒子形成BFC，它就能关闭住内部的浮动。此时最好的方法就是overflow:hidden属性。

清除浮动方法二：给后面的父盒子设置clear:both属性。clear表示清除浮动对自己的影响，both表示左右浮动都清除

清除浮动方法三：使用::after伪元素给盒子添加最后一个子元素，并且给::after设置clear:both

清除浮动方法四：在两个父盒子之间“隔墙”，隔一个携带clear:both的盒子

## 定位

### 相对定位

position:relative

盒子可以相对自己原来的位置进行位置调整，称为相对定位。

位置描述词

- left 向右移动
- right 向左移动
- top 向下移动
- bottom 向上移动
- 值可以是负数，即往规定方向相反移动

相对定位的性质，相对定位的元素，会在老家留坑，本质上仍然是在原来的位置，只不过渲染在新的地方而已，渲染的图形可以比喻成“影子”，不会对页面其他元素产生任何影响

相对定位的用途

- 相对定位用来微调元素位置
- 相对定位的元素，可以当作绝对定位的参考盒子

### 绝对定位

绝对定位：盒子可以在浏览器中以坐标进行位置精准描述，拥有自己的绝对位置。

position:absolute

- 绝对定位的元素脱离标准文档流，将释放自己的位置，对其他元素不会产生任何干扰，而是对它们进行压盖
- 脱离标准文档流的方法：浮动、绝对定位、固定定位
- 绝对定位的盒子并不是永远以浏览器作为基准点
- 绝对定位的盒子会以自己祖先元素中，离自己最近的拥有定位属性的盒子，当作基准点。这个盒子通常是相对定位的，所以这个性质也叫做子绝父相

#### 绝对定位的盒子垂直居中

绝对定位的盒子垂直居中是一个非常实用的技术

```
position: absolute;
top: 50%
margin-top: -自己高度一半;



p{
	width:80px;
	height:80px;
	background-color: orange;
	position: absolute;
	top: 50%
	left: 50%;
	margin-top:-40px;
	margin-left:-40px;
}
```

#### 堆叠顺序z-index属性

- z-index属性是一个没有单位的正整数，数值大的能够压住数值小的

#### 绝对定位的用途

- 绝对定位用来制作“压盖”、“遮罩”效果
- 绝对定位用来结合CSS精灵使用
- 绝对定位可以结合JS实现动画

### 固定定位

- 不管页面如果卷动，它永远固定在那里
- 固定定位只能以页面为参考点，没有子固父相这个性质
- 固定定位脱离标准文档流
- 固定定位用途：“返回顶部”、“楼层导航”

## 边框

### border属性

- border属性三个要素 `border: 1px solid red;`
- solid 实线  dashed 虚线  dotted  点状线



边框的三要素小属性

- border-width 线宽
- border-style 线型
- border-color 线颜色

四个方向的边框

border-top 上边框

border-right 右边框

border-bottom 下边框

border-left 左边框

border-left:none; 属性即可去掉左边框，依次类推。

三角形

```
.box {
	width:0;
	height:0;
	/*transparent透明色*/
	border:20px solid transparent;
	border-top-color: red;
}
```

### 	border-radius属性

- border-radius属性的值通常为px单位，表示圆角的半径
- border-radius属性可以单独设置四个圆角 左上、右上、右下、左下
- 正方形盒子如果设置的border-radius属性为50%，就是正圆形

### box-shadow属性

- border-shadow: 10px 20px 30px rgba(0,0,0,.4)  x偏移、y偏移、模糊量、颜色
- box-shadow属性值前加inset单词，表示内阴影
- box-shadow属性值可以用逗号隔开多个，表示携带多个阴影

## 背景颜色

### background-color属性

- background-color属性表示背景颜色
- 背景颜色可以用十六进制，rgb()，rgba()表示法表示
- padding区域是有背景颜色的

### background-image属性

- background-image属性用来设置背景图片，图片路径要写到url()圆括号中，可以是相对路径，也可以是http://开头的绝对路径
- 如果样式表是外链的，那么要书写CSS出发到图片的路径，而不是从html出发

### backgroud-repeat属性

- backgroud-repeat属性用来设置背景的重复模式
- repeat;   x，y均平铺 
- repeat-x; x平铺
- repeat-y; y平铺
- no-repeat; 不平铺

### background-size属性

- background-size属性用来设置背景图片的尺寸

contain和cover

- contain和cover是两个特殊的background-size的值
- contain表示将背景图片智能改变尺寸以容纳到盒子里
- cover表示将背景图片智能改变尺寸以撑满盒子 

### background-clip属性

- background-clip属性用来设置元素的背景裁切到哪个盒子
- border-box 背景延伸至边框（默认值）
- padding-box 背景延伸至内边（padding），不会绘制到边框处（仅在dotted、dashed边框可察觉）
- content-box 背景被裁剪至内容区

### background-attachment属性

- background-attachment属性决定背景图像的位置是在视口内固定，或者随着包含它的区块滚动。
- fixed 自己滚动条不动，外部滚动条不动
- local 自己滚动条动，外部滚动条动
- scroll 自己滚动条不动，外部滚动条动（默认值）

### background-position属性

- background-position属性可以设置背景图片出现在盒子的什么位置
- 可以使用top、bottom、center、left、right描述图片出现的位置
- CSS精灵：将多个小图标合并制作到一张图片上，使用background-position属性单独显示其中一个，这样的技术叫做CSS精灵技术，也叫作CSS雪碧图。

### background综合属性

```
background:white url(images/archer.png) no-repeat center center

背景颜色 背景图片 背景重复 背景位置
```

## 渐变

### 径向渐变

- 盒子的background-image属性可以用radial-gradient()形式创建径向渐变背景

  ```
  background-image:radial-gradient(50% 50%,red,blue)
  ```


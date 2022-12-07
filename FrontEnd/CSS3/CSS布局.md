# CSS布局

flex弹性布局

- 操作方便，布局极为简单，移动端应用广泛
- PC端浏览器支持情况较差
- IE11或更低版本，不支持或仅部分支持

## 布局原理

flex 是 flexible Box的缩写，意为“弹性布局”，用来为盒状模型提供最大的灵活性，任何一个容器都可以指定为flex布局。

- 当我们为父盒子设为flex布局以后，子元素的float、clear和vertical-align属性将失效
- 伸缩布局=弹性布局=伸缩盒布局=弹性盒布局=flex布局

采用Flex布局的元素，称为Flex容器（flex container），简称“容器”。它的所有子元素自动成为容器成员，成为Flex项目（flex item），简称“项目”。

**总结flex布局原理**

就是通过给父盒子添加flex属性，来控制子盒子的位置和排列方式。

## flex布局父项常见属性

### 常见父项属性

- flex-direction 设置主轴的方向
- justify-content 设置主轴上的子元素的排列方式
- flex-wrap 设置子元素是否换行
- align-content 设置侧轴上的子元素的排列方式（多行）
- align-items 设置侧轴上的子元素排列方式（单行）
- flex-flow 复合属性，相当于同时设置flex-direction和flex-wrap

### flex-direction设置主轴的方向

### 1、主轴和侧轴

在flex布局中，是分为主轴和侧轴两个方向，同样的叫法有：行和列、x轴和y轴

- 默认的主轴方向就是x轴方向，水平向右
- 默认的侧轴方向就是y轴方向，水平向下

### 2、属性值

#### flex-direction属性决定主轴的方向（即项目的排列方式）

注意：主轴和侧轴是会变化的，就看flex-direction 设置谁为主轴，剩下的就是侧轴。而我们的子元素是跟着主轴来排列的

- row 默认值从左到右
- row-reverse 从右到左
- column 从上到下
- column-reverse 从下到上

#### justify-content 设置主轴上的子元素排列方式

justify-content属性定义了项目在主轴上的对齐方式

*注意：使用这个属性之前一定要确定好主轴是哪个*

- flex-start 默认值 从头部开始 如果主轴是x轴，则从左到右
- flex-end 从尾部开始排列
- center 在主轴居中对齐（如果主轴是x轴则水平居中）
- space-around 平分剩余空间
- space-between 先两边贴边，再平分剩余空间（重要）

#### flex-wrap设置子元素是否换行

默认情况下，项目都排在一条线上（又称“轴线”）。flex-wrap属性定义，flex布局中默认是不换行的。

#### align-items 设置侧轴上的子元素排列方式（单行）

该属性是控制子项在侧轴（默认是y轴）上的排列方式，在子项为单项时候使用

- flex-start 默认值 从上到下
- flex-end 从下到上
- center 挤在一起居中（垂直居中）
- stretch 拉伸

#### align-content 设置侧轴上的子元素的排列方式（多行）

设置子轴在侧轴上的排列方式，并且只能用于子项出现换行的情况（多行），在单行下是没有效果的

- flex-start 默认值在侧轴的头部开始排列
- flex-end 在侧轴的尾部开始排列
- center 在侧轴中间显示
- space-around 子项在侧轴平分剩余空间
- space-between 子项在侧轴先分布在两头，再平分剩余空间
- stretch 设置子项元素高度平分父元素高度

#### align-items和align-content区别

- align-items 适用于单行情况下，只有上对齐、下对齐、居中和拉伸
- align-content 适用于换行（多行）的情况下（单行情况下无效），可以设置上对齐、下对齐、居中、拉伸以及平均分配剩余空间等属性值
- 总结就是单行找align-items  多行找align-content

#### flex-flow

flex-flow属性是flex-direction和flex-wrap属性的复合属性

```
flex-flow:row wrap;
```

## flex布局子项常见属性

- flex子项目占的份数
- align-self控制子项自己在侧轴的排列方式
- order属性定义子项的排列顺序（前后顺序）

### flex属性

flex属性定义子项目分配剩余空间，用flex来表示占多少份数

```
.item {
	flex: <number>; /* default 0 */
}
```

#### align-self 控制子项自己在侧轴上的排列方式

align-self属性允许单个项目有与其他项目不一样的对齐方式，可覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch。

```
span:nth-child(2) {
	/* 设置自己在侧轴上的排列方式  */
	align-self: flex-end;
}
```

#### order属性定义项目的排列顺序

数值越小，排列越靠前，默认为0

注意：和z-index不一样。




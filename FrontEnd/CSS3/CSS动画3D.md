# CSS动画3D

## 旋转变形

- 将transform属性的值设置为rotate()，即可实现旋转变形

  ```
  transform: rotate(45deg)
  ```

- 若角度为正，则顺时针方向旋转，否则逆时针方向旋转

## 缩放变形

- 将transform属性的值设置为scale(),即可实现缩放变形

- 当数值小于1时，表示缩小元素，大于1表示放大元素

  ```
  transform: scale(3);
  ```

### 斜切变形

- 将transform属性的值设置为skew()，即可实现斜切变形

  ```
  transform: skew(10deg,20deg)
  ```

  

### 位移变形

- 将transform属性的值设置为translate()，即可实现位移变形

  ```
  transform: translate(100px,200px); //向右移动 向下移动
  ```

- 和相对定位非常像，位移变形也会老家留坑，形影分离

## 3D旋转

- 将transform属性的值设置为rotateX()或者rotateY()，即可实现绕横轴、纵轴旋转。

  ```
  transform: rotateX(30deg); 
  transform: rotateY(30deg);
  ```

### perspective属性

- perspective属性用来定义透视强度，可以理解为“人眼到舞台的距离”，单位是px

### 空间移动

- 当元素进行3D旋转后，即可继续添加translateX()、translateY()、translateZ()属性让元素在空间进行移动
- 一定要记住，空间移动要添加在3D旋转之后

## 过渡基本使用

### transition过度

- transition过渡属性是CSS3浓墨重彩的特性，过渡可以在一个元素在不同样式之间变化自动添加“补间动画”

### transition属性基本使用

transition属性有4个要素

```
transition: width 1s linear 0s;
// 什么属性要过度  动画时长 变化速度曲线 延迟时间
```

所有数值类型的属性，都可以参与过渡，比如width、height、left、top、border-radius 

背景颜色和文字颜色都可以被过渡

所有变形（包括2D和3D）都能被过渡

过渡的四个小属性

- transition-property 哪些属性要过渡
- transition-duration 动画时间
- transition-timing-function 动画变化曲线（缓动效果）
- transition-delay 延迟时间

## 动画的定义

可以使用`@keyframes`来定义动画，keyframes表示“关键帧”，在项目上线前，要补上@-webkit-这样的私有前缀

```
@keyframes r {
	from {
	   transform: rotate(0);
	}
	to {
	   transform: rotate(360deg);
	}
}
```

### 动画的调用

定义动画之后，就可以使用animation属性调用动画

```
animation: r 1s linear 0s;
```

### 动画的执行次数

第五个参数就是动画的执行次数

```
animation: r 1s linear 0s 3;
```

如果想永远执行可以写infinite

```
animation: r 1s linear 0s infinite;
```

### 多关键帧动画

```
@keyframes changeColor {
	0% {
		background-color: red;
	}
	
	20% {
		background-color: yellow;
	}
	...
}
```


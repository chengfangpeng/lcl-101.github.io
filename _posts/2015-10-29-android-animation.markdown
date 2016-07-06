---

layout:     post

title:      "Android属性动画"

subtitle:   " \"Android Animation\""

date:       2015-10-29 12:00:00

author:     "X-ray"

header-img: "img/post-bg-2015.jpg"

tags:

    - Android

---









## 前言



Property Animation是Android3.0引入的一种功能强大的动画系统。它除了可以给普通的View添加动画效果外，还可以给对象添加效果。另外，Property Animation与Tween Animation一个最大的区别是Property Animation更改是对象的实际属性，而后者只是View的绘制效果，比如一个Button实现一个移动的动画效果，如果使用Tween Animation 的话，Button 的点击位置并不会随着动画的移动效果儿移动。换句话说在新位置Button是可能没有点击事件的。使用Property Animation可以设置下面的一些动画特性：

- Duration: 动画之间的间隔

- Time interpolation: 属性值的变化方式,可以表示为动画的事件函数，例如线性动画，加速动画等等。

- Repeat count and behavior: 动画的重复次数和重复方式。

- Animation sets: 动画集合

- Frame refresh delay: 帧刷新间隔，默认是10ms，但具体的速度依赖于系统的繁忙程度。



## 属性动画的工作原理


图１描绘了一个假象的对象x属性的动画，它给出了该对象在屏幕水平方法的位置，在40ms内改对象移动了40个像素。每10ms记录一次对象移动的像素，这个动画是的Time interpolation是liearinterpolation,表明动画是以恒定的速度移动的。



![图１](http://7xrrm5.com1.z0.glb.clouddn.com/blog_pic_article_property_animation_1.png)



在Property Animation中，ValueAnimator是其核心类，它记录了动画自身的一些属性值。图2是其工作流程：


![图２](http://7xrrm5.com1.z0.glb.clouddn.com/blog_pic_article_property_animation_2.png)



动画在整个过程中，会根据我们当初设置的TimeInterpolator和TypeEvaluator的计算方式计算出不通的属性值，从而不断的改变属性值的大小，就会产生各式各样的动画效果。

下面就通过一个实例理解一下什么是TimeInterpolator和TypeEvaluator.

- TimeInterpolator

TimeInterpolator翻译过来是插值器，插值器定义了动画变化中的属性变化规则，它根据时间比例因子计算出一个插值因子，那么什么是时间比例因子呢，简单讲就是：




```

时间比例因子　＝　动画已执行的时间　／　动画执行的总时间



```


而插值因子，用于设定动画是线性变化，还是非线性变化等千万中变化，你可以通过实现TimeInterpolator来实现自己的插值器(在>sdk22可以继承抽象类BaseInterpolator)。Android中默认的定义很多的插值器：

1. AccelerateDecelerateInterpolator

   在开始和结束时速度较慢

2. AccelerateInterpolator

   加速变化

3. LinearInterpolator

    匀速变化

更多的插值器,可以在[这里](https://developer.android.com/reference/android/animation/TimeInterpolator.html)查看.


 

```

//自定义插值器
class MyInterpolator implements TimeInterpolator{


    @Override
    public float getInterpolation(float input) {
        //自定义的规则
        return 0;
    }
}




```




- TypeEvaluator



TypeEvaluator是根据插值因子去计算属性值，Android默认可以识别的类型为int, float和颜色，分别是

IntEvalutor, FloatEvalutor, ArgbEvaluator.

1. IntEvalutor

    计算整数型

2. FloatEvalutor

    计算浮点型

3. ArgbEvaluator

    计算颜色属性




```



//自定义TypeEvaluator
class MyTypeEvaluator implements TypeEvaluator<Ball>{


    @Override
    public Ball evaluate(float fraction, Ball startValue, Ball endValue) {
        //自己的规则
        return null;
    }
}

```


下面以三个小球的旋转效果为例，了解一下属性动画的整个实现过程。





![效果图](http://7xrrm5.com1.z0.glb.clouddn.com/blog_pic_article_property_animation_3.png)



源码可以看[这里](https://github.com/chengfangpeng/BallView)

















    

































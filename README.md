# DSLAnimator

这是一个能让你在Kotlin下使用DSL的方式快速优雅的写Android动画的工具。
该项目是一个使用Kotlin的lambda表达式与扩展函数特性封装的DSL工具，如果你有一定的Kotlin基础，并且喜欢DSL的优雅，那么这个项目会是一个不错的学习对象。

## 一、它能做什么？

我们在写项目的时候，必不可少的需要经常为组件加上一些过渡动画，让其在与用户交互的时候可以更生动而不会显得乏味脱节。那么DSLAnimator能做的就是让你在写动画的时候逻辑更简单清晰，而不至于让大量的动画代码去混淆了其它业务内容的阅读体验。差不多就是我们把XML配置动画的方式搬到了Kotlin代码中，并且使用过程更加一体化。 DSLAnimator的唯一特点就是它让Android动画的开发变得更简单更清晰。

### 使用对比

![](https://raw.githubusercontent.com/550609334/DSLAnimator/master/images/DSLAnimator.gif)

##### 传统写法

上图的动画让一个按钮的宽高变大，同时有一个颜色的渐变动画，在我们平时用属性动画应该是下面这么写。

```kotlin
val width = ValueAnimator.ofInt(mButton.width, 500)
val widthParams = mButton.layoutParams
width.addUpdateListener {
      widthParams.width = it.animatedValue as Int
      mButton.layoutParams = widthParams
}

val height = ValueAnimator.ofInt(mButton.height, 500)
val heightParams = mButton.layoutParams
      height.addUpdateListener {
      heightParams.height = it.animatedValue as Int
      mButton.layoutParams = heightParams
}

val color = ValueAnimator.ofFloat(0f, 1f)
color.addUpdateListener {
      val value = it.animatedValue as Float
      val color = ArgbEvaluator().evaluate(value,
                  Color.parseColor("#636978"),
                  Color.parseColor("#1AD372")) as Int
      mButton.setBackgroundColor(color)
}

val set = AnimatorSet()
set.playTogether(width, height, color)
set.duration = 500
set.interpolator = OvershootInterpolator()
set.start()
```

##### 使用DSL

而使用了DSL的话，可以看到代码明显大量减少，并且逻辑更加清晰易懂。

```kotlin
animSet {
   widthAnim {
        target = mButton
        values = intArrayOf(mButton.width, 500)
   }
   heightAnim {
        target = mButton
        values = intArrayOf(mButton.height, 500)
   }
   colorAnim {
        target = mButton
        startColor = "#636978"
        endColor = "#1AD372"
   }

   duration = 500
   interpolator = OvershootInterpolator()
}.start()
```

## 二、怎么使用？

### 1、在build.gradle中添加依赖

```groovy
compile 'com.lzp.dslanimator:dslanimator:1.0.2'
```

### 2、简单使用

#### 1）单个动画

```kotlin
//像右移动100个像素
translationXAnim {
      target = mButton //要移动的控件
      values = floatArrayOf(0f, 100f)
      duration = 500
      interpolator = OvershootInterpolator()
}.start()
```

单个动画有返回一个NormalAnimator对象，可用其获取一些状态数据或对动画进行一些操作，例如可以回放动画

#### 2）集合动画

```kotlin
animSet {
   widthAnim {
        target = mButton
        values = intArrayOf(mButton.width, 500)
   }
   heightAnim {
        target = mButton
        values = intArrayOf(mButton.height, 500)
   }
   colorAnim {
        target = mButton
        startColor = "#636978"
        endColor = "#1AD372"
   }

   duration = 500
   interpolator = OvershootInterpolator()
   onEnd {
        Toast.makeText(this@MainActivity, "动画结束", Toast.LENGTH_LONG).show()
   }
}.start()
```

单个动画有返回一个Set对象，使用和NormalAnimator类似。

### 3、支持的动画类型

目前只支持属性动画，下表列出每个属性动画对应的DSL节点名

|     动画名      |       节点名        |
| :----------: | :--------------: |
|    alpha     |    alphaAnim     |
|    scaleX    |    scaleXAnim    |
|    scaleY    |    scaleYAnim    |
| translationX | translationXAnim |
| translationY | translationYAnim |
| translationZ | translationZAnim |
|   rotation   |   rotationAnim   |
|  rotationX   |  rotationXAnim   |
|  rotationY   |  rotationYAnim   |
|    width     |    widthAnim     |
|    height    |    heightAnim    |
|    color     |    colorAnim     |

### 4、支持的配置项

#### 1）单个动画

|     配置项      |                    描述                    |
| :----------: | :--------------------------------------: |
|    target    |               动画所要作用的View                |
|    values    | 动画所要运动的数值，与原属性动画一样。float类型给floatArray，int类型给intArray，例如floatArrayOf(0f, 100f)、intArrayOf(500) |
| repeatCount  |                和普通属性动画一样                 |
|  repeatMode  |                和普通属性动画一样                 |
|   duration   |                   持续时长                   |
|  startDelay  |                延迟多久后启动动画                 |
| interpolator |                   插值器                    |
|  startColor  |         开始的颜色值，只有colorAnim有这个配置项         |
|   endColor   |         结束的颜色值，只有colorAnim有这个配置项         |
|  onStart{}   |                 动画开始的回调                  |
|  onRepeat{}  |                 动画重复的回调                  |
|  onCancel{}  |                 动画被取消的回调                 |
|   onEnd{}    |                 动画结束的回调                  |
|  onUpdate{}  | 动画运行过程中，数值在更新的回调。这个lambda同样有回调一个ValueAnimator对象为参数。可提供变化的数值。 |

#### 2）集合动画

|     配置项      |                    描述                    |
| :----------: | :--------------------------------------: |
|   duration   |                   持续时长                   |
|  startDelay  |                延迟多久后启动动画                 |
| interpolator |                   插值器                    |
|   playMode   | 集合动画的播放模式，两种可选。并行播放PlayMode.TOGETHER，顺序播放PlayMode.SEQUENTIALLY，默认为并行。 |
|  onStart{}   |                 动画开始的回调                  |
|  onRepeat{}  |                 动画重复的回调                  |
|  onCancel{}  |                 动画被取消的回调                 |
|   onEnd{}    |                 动画结束的回调                  |
|  onUpdate{}  | 动画运行过程中，数值在更新的回调。这个lambda同样有回调一个ValueAnimator对象为参数。可提供变化的数值。 |

### 5、使用建议。

使用的时候其实可以先用一个懒加载在类的头部去先声明一个动画。然后在要用到的地方去调用。这样既不会浪费资源又能把动画代码放在最外面不去干扰业务，还能动态配置动画的值。

```kotlin
class MainActivity : AppCompatActivity() {

    private val anim by lazy {
        animSet {
            widthAnim {
                target = mButton
                values = intArrayOf(mButton.width, 500)
            }
            heightAnim {
                target = mButton
                values = intArrayOf(mButton.height, 500)
            }
            colorAnim {
                target = mButton
                startColor = "#636978"
                endColor = "#1AD372"
            }

            duration = 500
            interpolator = OvershootInterpolator()
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        mButton.setOnClickListener {
            anim.start()
        }
    }
}
```

### Email：[tracy550609334@gmail.com](mailto:tracy550609334@gmail.com)

License Copyright 2017 liu zipeng

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
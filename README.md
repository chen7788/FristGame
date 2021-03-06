有段日子没分享代码了,今天分享一个原本打算上架的项目,游戏做到中途本打算放弃开发,放了一段时间,觉得有些可惜,还是把主体功能写完成了,项目就不上架了,代码分享给大家吧.项目中的UI布局样式也是我自己设计的,图片有部分自己做的,有部分网上找的.如果觉得有兴趣上架的,可以自己完善下剩余的细节,自己发版吧.

发一部分图大家看下:(图片)
![aaa1.gif](https://upload-images.jianshu.io/upload_images/575247-97254f53b3e97abd.gif?imageMogr2/auto-orient/strip)
![aaa2.gif](https://upload-images.jianshu.io/upload_images/575247-40f1a9f389d44f51.gif?imageMogr2/auto-orient/strip)
![aaa3.gif](https://upload-images.jianshu.io/upload_images/575247-c544ee4100947d02.gif?imageMogr2/auto-orient/strip)

![aaa4.gif](https://upload-images.jianshu.io/upload_images/575247-9ab21d74def4e1c4.gif?imageMogr2/auto-orient/strip)

![屏幕快照 2018-04-03 15.09.25.png](https://upload-images.jianshu.io/upload_images/575247-ff1db85e0a3ceb37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

游戏是消除类的游戏,核心玩法是玩家通过拇指在屏幕绘制图形,当玩家绘制的图案与怪物身上气球的图案相同时,气球被销毁,如果怪物身上气球数为0时,怪物会被消灭.

#### 用户绘制图形样式(sameTouch)

项目开始前,首先要设计玩家可绘制的图形.我这里设计以下图案,图片的制作采用Photoshop,关于项目中UI的制作就不讲了,感兴趣的可以自学下PS,相关的教程网上一大片.这里也可以设计一些复杂的手势,随着游戏难度的提升而出现.需要注意的是,设计的手势不要太过相近,否则后面不好做判断.

怪物身上气球样式:
![pop1.png](https://upload-images.jianshu.io/upload_images/575247-270d1126d5139e3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![pop2.png](https://upload-images.jianshu.io/upload_images/575247-83108d4e06556a16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如何判断玩家绘制的图形是什么样式?

方案一:首先我想到的是图像识别功能.给定一个模型库装入设定好的可绘制图形样式.将玩家绘制的路径变成黑色,背景为白色,玩家每次绘结束后,获取玩家绘制的图片,并将图片尺寸缩放到与模型库图片大小相同,通过图形的对比,判断用户绘制图形与模型库中哪种图形最相近,并给出相似度,当相似度大于70%时,即认定玩家绘制的图形.然而理想很丰满,现实很骨感~首先每次获取玩家绘制路径并且生成图片,这种频繁的操作,手机能明显的感觉卡顿.并且为了做图像的对比,我又引入了opencv框架.一来一去,效率实在太慢了,so~放弃此方案.(opencv,一套强大图像处理的框架,这里使用有点杀鸡用牛刀了).

方案二:首先,将手机看做XY平面坐标,在屏幕上一一绘制开发者设定的标准图像,并且记录绘制路径的二维坐标数组,一套坐标数组对应一种图形.将所有设定好的图形坐标数组保存起来,当做模型库与后面玩家绘制的坐标数组对比.当玩家绘制好路径后,将玩家绘制的坐标点数组与当前库内的坐标数组一一对比,求出与之相近的图形,并且返回一个相近度,根据图形复杂程度与相近度的结合,判断用户绘制的图形.具体代码看工程中的sameTouch文件下的代码.这个方案在速度与效率上是可行的,但也有弊端.由于这个方案是根据路径中坐标点差值的计算而判断的,实际上一个图形会有多种绘制方案,比如数字8,有人是从由上至下开始绘制,有人是从中间到上再到下绘制,还有人是中间到下再到上绘制,不同玩家会有不同的书写习惯,但是最终得到的图形结果都应该是8,这样就需要模型库中有同一个图像的多套模型路径数组.(目前项目中采用的就是这种方案).

方案三:这个方案是我后来研究机器学习时候发现的方案,并且可以确定,这个方案更适用这个项目.就是采用神经网络方案,通过坐标组来判断,由于项目搁浅了,就没有换成这个方案.感兴趣的朋友可以研究下.附上神经网络的链接,就是采用里面的方案,我自己测试了下数字0-9,非常准确,推荐使用此方案.[神经网络](http://wiki.jikexueyuan.com/project/tensorflow-zh/)

至此,游戏中最核心的功能得以实现,剩下的就是添砖加瓦了.

#### 首页(HomeScene)

首页布局没啥可说的,就是一些按钮+图片构成,采用cocos studio布局,通过加载csb文件.

下面三个小怪物自己会有默认动画播放,当玩家点击也会有交互动画.

游戏分为两种模式,一种是计时模式,一种是守护模式.

*   计时模式:游戏的时间为固定模式(在游戏开始前可以通过购买游戏道具增加时间).时间到了,游戏结束.

*   守护模式,玩家有固定生命值(在游戏开始前可以通过购买游戏道具增加生命值),每当一个怪物miss后,生命值减一,当生命值为0,游戏结束.这里加了个功能,随着游戏进程越长,怪物下落的时间也会越快.

玩家可以通过游戏中获取金币或者直接购买金币,金币的作用是在游戏开始前,可以购买道具,增加游戏的娱乐性.

#### 游戏场景(GameScene)

按着游戏的内容,场景主要分为四大层:

*   GameControlLayer:用于游戏数据的展示,如获取金币数量,击杀怪物数量,连杀特效.同时也兼顾暂停,结束等UI的展示.这里也同样都可以在单独拆分出来,看开发者自己的习惯吧.

*   MonsterLayer:用于控制游戏怪物的层级.游戏中怪物的出现,下落,气球的销毁,怪物销毁等.

*   UserDrawLayer:用于记录玩家绘制图形的层级,每次绘制完成后,将玩家绘制图形的结果传给MonsterLayer,判断是否有可供消除的气球.

*   GamePropLayer:道具层,道具分为2种可使用道具一种被动道具,可使用的为炸弹(消灭当前所有怪我).减速道具:较少怪物下落速度.

就写这些吧,游戏比较简单,开发起来也不算困难,感觉没啥需要可写的,耗时最多的都在UI的制作上啦.有兴趣的朋友可以下载代码自行研究下.

代码地址:

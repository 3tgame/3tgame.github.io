---
layout: post
title:  "地图出现黑线、图片边缘被切掉、图片模糊的原因及解决方法"
date:   2014-01-09 03:01:18
categories: [Cocos2dx]
---

一、地图出现黑线

原因、解决方法 可参看 [How To Fix Artifacts With Cocos2D](http://rainbowcoding.com/how-to-fix-artifacts-with-cocos2d/)。

原因

绘制时某些拷贝操作失败而导致拷贝了隔壁其他Sprite的某一部分，导致出现其他色值的线条。

解决方法

方法1： 在使用TexturePacker打包时，使用Extrude属性。如TexturePacker *.png --extrude 1 --plist test.plist --sheet test.png

方法2： 另一种不太好的方式是修改ccConfig.h，把#define CC_FIX_ARTIFACTS_BY_STRECHING_TEXEL 0
修改为#define CC_FIX_ARTIFACTS_BY_STRECHING_TEXEL 1。

说它不好原因可看ccConfig.h的注释
     {% highlight c++ %}
If enabled, the texture coordinates will be calculated by using this formula:
- texCoord.left = (rect.origin.x*2+1) / (texture.wide*2);
- texCoord.right = texCoord.left + (rect.size.width*2-2)/(texture.wide*2);

The same for bottom and top.

This formula prevents artifacts by using 99% of the texture.
The "correct" way to prevent artifacts is by using the spritesheet-artifact-fixer.py or a similar tool.
     {% endhighlight %}
修改后，它只会使用原来图片的99%，故图片的边缘像素会被裁掉。
且它的影响是全局的，对CCSprite、CCSpriteBatchNode、CCLabelAtlas等都有影响。

方法3：使用setAliasTexParameters()关闭抗锯齿（cocos2dx对纹理默认是开启抗锯齿）。

二、图片边缘被切掉

现象

圆形边框变得不圆，仔细看会感觉边缘的像素被切掉。

原因

使用上文介绍的地图出现黑线的第2个解决方法。

解决方法

不要使用上文介绍的地图出现黑线的第2个解决方法，改用第1种方法；
或在CCSprite新写一个方法包含当#define CC_FIX_ARTIFACTS_BY_STRECHING_TEXEL 1时执行的逻辑，对于不允许出现黑线的图片（如地图）调用该新写的函数。

三、图片模糊

1. 图片模糊透明

原因：

纹理在初始化的时候默认调用了setAntiAliasTexParameters()接口，而该接口设置GL_TEXTURE_MIN_FILTER和GL_TEXTURE_MAG_FILTER为GL_LINEAR。而使用GL_LINEAR渲染时，如果文理的像素点和显示区域的坐标不能一一对应（例如放大，缩小，或者显示位置为浮点数），就会使用附近的四个像素（2*2， 2D显示）做颜色混合，这样会使图片看上去模糊。

解决方法：

(1). 调用Texture的setAliasTexParameters接口，副作用在放大或缩小时会导致锯齿严重，而且如果显示位置为浮点数的话，会导致最后一行（一列）像素被截掉。

(2). 不要使用图片分辨率为奇数的图片，因为使用奇数的图片，如果设置AnchorPoint为(0.5f, 0.5f), 即使setPosition()传入整数参数，也会导致图片的显示坐标为小数。

(3). 如果一定要使用分辨率为奇数的图片，锚点设为(0.0f, 0.0f), 这样可以保证只要setPosition()传入的是整数(注意父控件的位置也不能是浮点数)就不会有显示问题。

2. 图片感觉模糊不清晰

原因:

引擎默认使用的是透视投影模式，此模式的效果为近大远小的效果，所以远处的则模糊不清晰

解决方法：

可以使用 CCDirector::sharedDirector()->setProjection(kCCDirectorProjection2D); 设置引擎为正交投影模式。


参见：

[【iOS-Cocos2d游戏开发之十八】解决滚屏背景/拼接地图有黑边(缝隙)/图片缩放后模糊透明/图片不清晰【2013年12月13日补充】/动画播放出现毛边以及禁止游戏中自动锁屏问题！](http://www.himigame.com/iphone-cocos2d/507.html)

[cocos2d-x如何解决图片显示模糊问题 ](http://blog.csdn.net/yuanhong2910/article/details/7506593)

[cocos2d-x中使用CCSprite拼接有缝隙](http://blog.csdn.net/wolfking_2009/article/details/8704526)

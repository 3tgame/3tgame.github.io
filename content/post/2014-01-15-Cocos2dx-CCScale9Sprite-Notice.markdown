---
layout: post
title:  "CCScale9Sprite 不支持含透明像素的图片的缩放"
date:   2014-01-15 01:01:18
categories: [Cocos2dx]
---

kongbai.png是包含20\*20个透明像素的图片，使用TexturePacker打包后，在最终纹理那里只有1\*1个像素。

使用 local img = CCScale9Sprite::createWithSpriteFrameName("base/kongbai.png", CCRect(10,10,1,1));

创建九宫图片后，会发现创建非透明的图片，现象是会显示一些乱七八糟的纹理。

CCScale9Sprite中调用bool CCScale9Sprite::updateWithBatchNode(CCSpriteBatchNode* batchnode, CCRect rect, bool rotated, CCRect capInsets)
设置9个CCSprite的纹理区域信息。

该函数中使用如下语句计算各个CCSprite所用纹理区域的大小（注释为上面例子的计算）：
 {% highlight c++ %}
    float left_w = m_capInsetsInternal.origin.x;                 // 10
    float center_w = m_capInsetsInternal.size.width;      // 1
    float right_w = rect.size.width - (left_w + center_w); // 1 - (10 + 1) = -10
    float top_h = m_capInsetsInternal.origin.y;                // 10
    float center_h = m_capInsetsInternal.size.height;          // 1
    float bottom_h = rect.size.height - (top_h + center_h);     // 1 - (10 + 1) = -10
{% endhighlight %}
如果kongbai.png在打包后纹理的位置为(354, 790);
经过矩阵转换后，
 {% highlight c++ %}
     leftcenterbounds.origin = {x=354.00000 y=800.00000 }
     leftcenterbounds.size = {width=10.000000 height=1.0000000 }
     centerbounds.origin = {x=364.00000 y=800.00000 }
     centerbounds.size = {width=1.0000000 height=1.0000000 }
     rightcenterbounds.origin = {x=355.00000 y=800.00000 }
     rightcenterbounds.size = {width=10.000000 height=1.0000000 }
     ...
{% endhighlight %}
注意上面的right_w、bottom_h的结果为负数，故导致算出来的边界有点奇怪，rightcenterbounds的范围与leftcenterbounds和centerbounds的范围重叠。且leftcenterbounds的范围已超出kongbai.png在大纹理中占的1\*1的区域，故会导致使用隔壁纹理作为显示内容，显示一些缩放后的其他图片的部分区域。

解决方法：

1、不要使用透明像素的图片

2、如果只包含透明像素，可设置capInsets时，把原始图片当做1*1来设，或不设capInsets，会自动使用CCRectMake(w/3, h/3, w/3, h/3)作为capInsets。

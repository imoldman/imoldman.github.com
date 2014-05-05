---
layout: post
category : tip
title : 如何让自定义动画支持慢速播放(iOS Simulator)
tags : [c++, objective-c, ios]
---

{% include JB/setup %}

<link rel="stylesheet" type="text/css" href="{{ root }}/css/pygments/native.css" />

## 前言

近来[`pop`](https://github.com/facebook/pop)盛行，我也不能免俗，跑去学习了一遭。`pop`的动画机制和使用方式自有[其他人来解释和说明](http://weibo.com/1659808677/B2RGslf3J)，不需要我再罗列一遍。这里我说一个比较有意思的东西。
___

## 阐述

我一向对调试支持比较感兴趣， 在`pop`的文档中，如下的说明一下子吸引了我。

	Pop obeys the Simulator's Toggle Slow Animations setting. Try enabling it to slow down animations and more easily observe interactions.

到底是怎么做到的呢？源码之前，了无秘密。

{% highlight objc %}
#if TARGET_OS_IPHONE
#import <UIKit/UIKit.h>
#endif

#if TARGET_IPHONE_SIMULATOR
UIKIT_EXTERN CGFloat UIAnimationDragCoefficient(); // UIKit private drag coeffient, use judiciously
#endif

CGFloat POPAnimationDragCoefficient()
{
#if TARGET_IPHONE_SIMULATOR
  return UIAnimationDragCoefficient();
#else
  return 1.0;
#endif
}
...
- (void)render
{
  CFTimeInterval time = CACurrentMediaTime();

#if TARGET_IPHONE_SIMULATOR
  // support slow-motion animations
  time += _slowMotionAccumulator;
  float f = POPAnimationDragCoefficient();

  if (f > 1.0) {
    if (!_slowMotionStartTime) {
      _slowMotionStartTime = time;
    } else {
      time = (time - _slowMotionStartTime) / f + _slowMotionStartTime;
      _slowMotionLastTime = time;
    }
  } else if (_slowMotionStartTime) {
    CFTimeInterval dt = (_slowMotionLastTime - time);
    time += dt;
    _slowMotionAccumulator += dt;
    _slowMotionStartTime = 0;
  }
#endif

  [self renderTime:time];
}
{% endhighlight %}

不难发现，这里的关键就是一个名叫`UIAnimationDragCoefficient`的私有API，通过[`iphonedevwiki`](http://iphonedevwiki.net/index.php/UIViewAnimationState)了解到它的解释是这样的。

	The drag coefficient is a multiplier applied on time measurements. A large drag coefficient can slow down animations.
	
	The drag coefficient is obtained with the UIAnimationDragCoefficient() function, which in turn is an integer of the key UIAnimationDragCoefficient in the preference file ~/Library/Preferences/com.apple.UIKit.plist.
	
	Drag coefficient will not affect non-UIKit animations.
	
所以，显而易见，`pop`借用了这个`API`来实现本来不会受模拟器慢速播放影响的动画。

真是有意思的`API`。另外`iphonedevwiki`是一个越狱开发的`wiki`，里面有很多实用的私有API列表，比如`UIView`内的[这个方法](http://iphonedevwiki.net/index.php/UIView#-recursiveDescription)。

{% highlight objc %}
-(NSString*)recursiveDescription;
{% endhighlight %}

它可以递归输出某`view`的层次关系，如：

{% highlight objc %}
<UIWebView: 0x4116bb0; frame = (0 100; 320 230); layer = <CALayer: 0x4116c20>>
   <UIScroller: 0x411e110; frame = (0 0; 320 230); autoresize = H; layer = <CALayer: 0x411e4d0>>
       <UIImageView: 0x411f460; frame = (0 0; 54 54); layer = <CALayer: 0x411f490>>
       <UIWebDocumentView: 0x4812c00; frame = (0 0; 320 230); layer = <UIWebLayer: 0x41171c0>>
{% endhighlight %}

我经常用这个技巧来查阅一些控件，会有意向不到的收获。
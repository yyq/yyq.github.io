---
layout: post
title: "cs193p HW1 所做所看所想"
date: 2013-06-28 18:37:01
comments: true
tags: [iOS,objective-c]
---
花了两个礼拜的时间，看完了cs193p的前面三堂课，并且做完了作业。没搞明白的东西，在piazza上发问，很快有人回答我解决的问题，感觉真好。

做完hw1之后，平时总是喜欢招人讨论的我，这下好像在周围找不到同学讨论了。于是就在piazza上闲逛，看看别人的问题和别人的回答。

论坛中，有加粗的**I**字母标记的都是有被导师回答的问题。于是我就挨个把导师参与过的讨论都看了一遍。看到某一个，觉得很赞，一定要在博客中写下来。

## 收获1：关于思想
不是关于某种技术方案，也不是关于某个细节技巧。而是关于思想。
<!--more-->
#### hw1的背景如下：
课堂上老师完成一本基本的翻纸牌的游戏，一句话来说他的规则就是，当你翻到的两个纸牌数字相同或者花色相同就OK就可以得分什么的。在作业1中有一个拓展，就是说你得自己写一个三个纸牌的游戏的object来。

#### 问题背景如下：
既然有two-card-game,作业里也有拓展要求three-card-game，why don't we just implement A N-card-game？原文问题摘抄如下：

>Just wondering what everyones thoughts are on the best way to implement both a 2 and 3 card game from an OO perspective.

>The obvious approach might be to create a CardMatchingGame superclass (e.g. turn the existing CardMatchingGame class into an abstract class), and two concrete subclasses TwoCardMatchingGame and ThreeCardMatchingGame that override the flipCardAtIndex method to do the "right" thing in each case.

>Alternatively it would be easy to have a property in the CardMatchingGame class which stores the game type, and then case the logic in flipCardAtIndex according to this property.

>As another alternative would be to write the function flipCardAtIndex in a more generic fashion which works for an n-card flipping game (and perhaps do the same in the PlayingCard match function).

>Any thoughts?

>Thanks,
>Adam

#### 我的想法1
我一看到这个问题，觉得也是，这想法不错。我做一个N个纸牌的游戏的方案来，N也就是一个参数罢了。就算是一个完全的解决方案了，多好的想法！

在看完Instructor的答案之后，我才意识到，这么好的想法是多么的学院派。

#### 老师的答案（原文摘抄）
>Great questions. A lot of this really boils down to your specific style and the specific needs of your program, because I don't think there's really a prescribed answer here - each of the options you proposed embody certain trade offs, and this type of OOP design can really boil down to an art more than an exact science.

>Personally, in this type of situation I'm a fan of making abstract classes, because it separates the code and its easy to layer on features without worrying about breaking old code that works. On the other hand, making one class that supports multiple game types and writing methods generically without hard coding assumptions about the game are also great tactics, and its really just a matter of style.

>Keep in mind though, if you make things too generic it can be easy to over complicate things quickly and end up with a lot of extra code that you really didn't need. Try to strike a balance between getting things working and anticipating how you might want to expand the functionality of your program later.

#### 我的想法2
看过老师的答案之后，感觉吧，不得不信服人家所说的。这个答案的第三段话我感触最深：如果你做一个很抽象的东西，它可能可以很好的适应复杂的参数等，但是也会因此附带上一大堆其他的其实你可能并不需要的代码。得尝试找到一个平衡，在完成working任务和预测你之后将要花费多大功夫去扩展新的功能。


## 收获2：关于细节

关键函数：**setImage:forState**,**UIControlState**

废话不多说，直接看我贴的我和别人的对话吧。我提的问题，别人回答。

**Answer:**
>cardButton setImage:(card.isFaceUp ? nil : cardBackImage) forState:UIControlStateNormal];

**Ask**
>So . I forget the  nil pictures and faceup things. 
Because, I thought like this: If the state is Normal , just show the backImage.  How  the faceup effect on it? Maybe I still don't understand what the   UIControlStateNormal  mean ? 
could you tell me more about it?

**Answer**
>[link](http://developer.apple.com/library/ios/#DOCUMENTATION/UIKit/Reference/UIControl_Class/Reference/Reference.html#//apple_ref/doc/c_ref/UIControlState)
 
>UIControlStateNormal
The normal, or default state of a control—that is, enabled but neither selected nor highlighted.
 
>The button will be in the normal state both when the card is face up but also if the card is face down.
The only time the button is in selected and/or highlighted is when we tap on the button and the card is being 'flipped over'.

**Ask**
>I  only  set the  backImage for the normal state.
and I  didn't set  any other images for the other states.
Why when I selected the button. The button is not in the NormalState , but still  show me the backImage ?

**Answer**
> Great question, and the answer is given [here](http://developer.apple.com/library/ios/#DOCUMENTATION/UIKit/Reference/UIButton_Class/UIButton/UIButton.html#//apple_ref/occ/instm/UIButton/setImage:forState:)
 
>In general, if a property is not specified for a state, the default is to use the UIControlStateNormal value. If the UIControlStateNormal value is not set, then the property defaults to a system value. Therefore, at a minimum, you should set the value for the normal state.


#### 细节决定成败，这说法不是盖的。可能我在看英文文档的时候，对这样的句子并不是很敏感，看了函数的简单用法就直接略过了。而别人却一眼就能看穿我的问题所在。以后看英文文档可不能太随意了。










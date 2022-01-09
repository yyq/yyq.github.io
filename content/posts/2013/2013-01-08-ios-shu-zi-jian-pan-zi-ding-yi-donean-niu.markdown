---
layout: post
title: "ios 数字键盘自定义done按钮"
date: 2013-01-08 11:36:01
comments: true
tags: [ios,xcode,NumberPad]
---
总体逻辑，监听系统数字键盘的弹出和消失，在系统键盘弹出时，show my self-defined *done key*,在系统键盘消失时，hide it

<!--more-->

## 1.界面上的操作

 1. 在stroyboard中选中某个按钮，然后option+command+4，调出*Attributes inspector*,在*test field*里面的keyboard，得用**Number Pad**，如下图：
 ![x](2013-01-11-9.31.55.png)
 
 1. 按住control，把按钮拖到viewcontroller.h文件，选择*Connection*为**Action**,*Event*为**Touch Down**,对应的name就自己随意取个名字好了。
 
## 2.添加代码

### 响应touch down事件的代码：
	- (IBAction)NOBut:(id)sender {
    	doneInKeyboardButton.hidden=NO;
	}
这里我的事件的名字是用的NOBut。doneInKeyboardButton就是那个按钮了。接下来我们要添加的是这样两个部分，**1**让这个自定义按钮在numberpad弹出的时候弹出；**2**让这个自定义按钮在numberpad消失的时候也消失。

### 注册系统键盘弹出和消失的通知
在viewDidLoad的函数里添加：
	    
	[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(handleKeyboardDidShow:) name:UIKeyboardDidShowNotification object:nil];
    
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(handleKeyboardWillHide:) name:UIKeyboardWillHideNotification object:nil];
    
### 反注册通知
在dealloc函数里添加
	
	[[NSNotificationCenter defaultCenter] removeObserver:self];
在我参考的博客里，人家还写了一句什么[**** release];?

但是我这个工程好像是用了ARC，自动retain计数？就不用自己管理内存空间了？而且有了那句话还会报错，我就直接把那个super release的代码去掉了。

恩，接下来我们写*handleKeyboardDidShow*函数和*handleKeyboardWillHide*函数，就是让自定义的那个按钮出现和消失的函数。

### 弹出该按钮的函数
	
	- (void)handleKeyboardDidShow:(NSNotification *)notification
	{
		//如果这个按钮不存在，那就来一个
    	if (doneInKeyboardButton == nil)
    	{	//自定义按钮
        	doneInKeyboardButton = [UIButton buttonWithType:UIButtonTypeCustom];
        	//根据不同手机屏幕，把按钮放在不同的位置
        	CGFloat screenHeight = [[UIScreen mainScreen] bounds].size.height;
        	if(screenHeight==568.0f){//爱疯5
            	doneInKeyboardButton.frame = CGRectMake(0, 568 - 53, 106, 53);
        	}else{//3.5寸
            	doneInKeyboardButton.frame = CGRectMake(0, 480 - 53, 106, 53);
        	}
        
        	doneInKeyboardButton.adjustsImageWhenHighlighted = NO;
        	//加载图片，这里引用其他的文章里的内容了：图片直接抠腾讯财付通里面的= =!
        	[doneInKeyboardButton setImage:[UIImage imageNamed:@"btn_done_up@2x.png"] forState:UIControlStateNormal];
        	[doneInKeyboardButton setImage:[UIImage imageNamed:@"btn_done_down@2x.png"] forState:UIControlStateHighlighted];
        	//如果触发了这个按钮的TouchUpInside事件，则开始finishAcion的操作
        	[doneInKeyboardButton addTarget:self action:@selector(finishAction) forControlEvents:UIControlEventTouchUpInside];
    	}
    	//显示出这个按钮
    	UIWindow* tempWindow = [[[UIApplication sharedApplication] windows] objectAtIndex:1];
    	//恩，这里这一行代码什么意思，我还没看懂，若有人理解，请在博客下方留言，谢谢
    	if (doneInKeyboardButton.superview == nil)
    	{
        	[tempWindow addSubview:doneInKeyboardButton];    // 注意这里直接加到window上
    	}
	}
### finishAction事件的内容：关闭键盘
	-(void)finishAction{
    	[[[UIApplication sharedApplication] keyWindow] endEditing:YES];//关闭键盘
	}


### 消失该按钮的函数
	- (void)handleKeyboardWillHide:(NSNotification *)notification
	{
    	if (doneInKeyboardButton.superview)
    	{
        	[doneInKeyboardButton removeFromSuperview];
    	}
	}
	
### all right , the **Done** Button is done
	
	
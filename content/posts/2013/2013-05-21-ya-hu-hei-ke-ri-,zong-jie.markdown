---
layout: post
title: "第一次参加Hackthon的活动:2013 Yahoo China HackDay"
date: 2013-05-21 15:38:01
comments: true
tags: [mashup,yahoo,hackthon,flickr,YQL]
---
上个周末，得瑟了整整30多个小时吧。周六早8点集合，12点正式开始比赛。周日12点比赛结束，下午2点开始展示，5点钟颁奖。200来自全国各地的同学参赛，45个队伍。6个获奖队伍，可惜没有我。

认识了新的朋友（北邮的李博洋，郭冰，清华的刘李洋），参观了雅虎北京研发中心的办公室，体验了雅虎公司的丰富的早饭中饭晚饭和自助餐

第一次参加通宵编程的比赛，人生又更加完美了一些！

哦，对了，我还收获了一个纪念奖（证书，瓷杯，T-shirt）。

完后还给YQL在github上的table提交了一个pull request:[Link](https://github.com/yql/yql-tables/pull/367)

### 请注意：以下内容为意识流
<!--more-->

### 1.认识新朋友
之前就在雅虎通知的网站上，自寻队友，找队友没有什么目的性，好像只要像是北京的我都接受，于是很快通过邮件和QQ联络到了另外3个人，我们4个人就是一队了。在QQ上简略的聊了一些比赛的思路，考虑mashup类开发等等。

礼拜六早上8点，我匆忙赶到在东主楼实验室拿点参考资料，然后call刘李洋同学（清华电子系大三）在主楼国旗那儿见面，很清瘦的小子，他跟我说他懂得计算机不是很多，目的就是为了找实习，所以四处投简历。可是他好像最近人品很背：昨天刚刚装完ubuntu，grub坏了进不了系统了，早上出门时候发现单车坏了，我跟他说你走路过来吧，后来才知道他宿舍在紫荆那边，他最后用身份证抵押借了一辆单车骑到主楼这边。

到了清华同方大厦11层，联系了另外两名队友，很快便算是认识了，北邮的两个同学都和我一样，研二计算机的，李博洋一副拖鞋外加白衬衫，一看就像是技术牛人似的（从之后的聊天和活动中看出，确实牛），郭冰同学很热心，立马就拿着饮料给我和刘李洋端了过来。稍微聊天之中，让我最惊讶的一件事情是：我问郭冰和李博洋他们两在实验室的研究方向，他们回答的简练干脆：**我们实验室没有研究方向，老师有什么样的项目，我们就做什么，我们什么技术都接触过一些**。我又问那你们毕业设计的方向呢？也没有么？**那就看到时候老师有什么项目了，项目当毕设**

这着实让我非常的惊讶，看看自己平常的实验室打酱油的日子，看似跟老师学习做研究的日子，感觉有些荒废，按部就班的按照老师的指导，看看论文，总结总结，随意写点代码，反正也没有人催促我赶进度。对自己的研究生生涯略感失望。。。或许很多年之后才能体会到，以找工作为目的的研究生应该搞点学术研究好还是搞点工程项目好。。

### 2.开始比赛了
上午听人家扯淡了好几个小时，12点钟我们组找了个座位，然后吃过快餐，开始了讨论。
看过了一圈yahoo challenge,觉得吧，有几个又太简单，有几个又太难，不是很擅长，果断放弃了yahoo challenge。

讨论出结果是做一个地图类的web-app应用,mashup类，综合地图，推荐饭馆酒店，图片，天气，留言等信息；迅速开始了分工。刘李洋和郭冰开始分别调查google maps的API和百度地图的API,李博洋开始着手搭建服务器和爬相关饭店酒店数据，我调查研究雅虎方面的API（flickr和天气）。

半夜2点的时候，郭冰同学已然困了，我们3个没有管它。
我基本搞定了flicker 和 weather的API工作，在深入研究
李博洋已经开始了在基于百度地图的web-app上添加自己的评论系统
刘李洋在重装系统之后，开始了研究图像中人脸识别算法的实现。已经找到opencv的代码可以跑出结果来，他一直在优化程序，就差和 web-app集成了。本来是预计在app中根据地理位置拿到附近的所有flickr的图片，然后增加一个特殊功能是，只给出用户带有人脸的图片这样一个功能。可以最后在移植opencv程序从win到linux遇到了一些困难。然后又木有人擅长C++的socket程序（预计可以服务器给这台电脑picture的url,然后计算完是否有人头之后，通过socket返回结果即可）。
本来能成为一个亮点的，结果黄了。


# 3.这一段才是真正的技术博客
本来是想写技术博客的，结果愣是被我写成了流水账。这一段写一写我学习到的技术吧.
我重点学习使用了yahoo的YQL和flickr的API。

YQL真心强大无比，我理解这个东西为网络数据库？只需简单的几句话发送给yahoo，就能从yahoo或者其他第三方拿到自己想要的数据：例如flickr照片，地理位置，天气等等。更可以根据yahoo提供的格式，自己定制自己想要的table的样式,然后再从table中拿数据。

flickr的API的链接：<http://www.flickr.com/services/api/>,我在这次比赛中用到的几个主要的API是flickr.photo.search来搜索不同条件的图片，还有用到相关的地理位置信息等。

然后YQL的最强大之处在于，Yahoo给出的web console,直接就可以测试自己写的YQL语句的结果：<http://developer.yahoo.com/yql/console/>.敲完YQL句子，一回车就能看到返回结果。

下面给出几个我在比赛中用到的句子吧：


#### 3.1从地理位置找图片
 
		select * from flickr.photos.search 
		where has_geo="true" and lat="39.9969808" and lon="116.3323017" 
		and api_key="92bd0de55a63046155c09f1a06876875";

上面这个句子就是说在flickr的图片库里，找寻地理位置（经度维度为中心，某个半径）的图片，其实这个search的API,还有无数的多得难以想象的参数可供设置，具体内容看看[这里](http://www.flickr.com/services/api/flickr.photos.search.html)。

在YQL返回了结果之后，就可以通过其中的字符串计算出图片的url了，参考yahoo的博文如下：[Building Flickr URLs from YQL flickr.photos.search results
](http://developer.yahoo.com/blogs/ydn/building-flickr-urls-yql-flickr-photos-search-results-7921.html)

#### 3.2从图片ID找图片的拍照时间（需求：按照拍照时间给照片排序）上一步工作的返回值，已经有了photo的ID,这一步就是从ID到拍照时间。
  	
从flickr的API中可以获取很多很多图片的信息的，但是从photos.getinfo的API,必须之前通过auth验证等，获得权限才能有这一步操作，不然就返回table not found这样的错误。短时间内authAPI我就没有去研究了。然后在YQL的文档页面找到了可以创建自己的表，然后从自己的表中拿出响应的内容就可以。我参考了一些github上的YQL的开源的flickr的table，然后自己写了个一个xml文件作为自己的table,看上去可以绕过权限？，反正结果是它不报错说table not found了。参考链接：[Link](https://github.com/yql/yql-tables),然后我自己写了一个如下的xml文件：源码放在了[这里](https://github.com/yyq/yahoo)
```
	<?xml version="1.0" encoding="UTF-8"?> 
	<table xmlns="http://query.yahooapis.com/v1/schema/table.xsd">  
				<meta> 
    		<author>David Young;davidbeckham2901@gmail.com</author>  
    		<documentationURL>http://www.flickr.com/services/api/flickr.photos.getInfo.html</documentationURL>  
    		<sampleQuery>select * from {table} where photo_id="109722179"</sampleQuery>  
		</meta>  
		<bindings>  
			<select itemPath="rsp.photo.dates" produces="XML">  
      			<urls>  
        			<url env="all">http://api.flickr.com/services/rest/?method=flickr.photos.getInfo</url>  
      			</urls>  
      			<inputs>  
        			<key id="photo_id" type="xs:string" paramType="query" />  
        			<key id="api_key" type="xs:string" private="true" paramType="query" default="4fb031bf5b2f138576d011ff37f31565"/>   
      			</inputs>  
    		</select>  
		</bindings>  
	</table>
```	
   其中用的API其实还是getInfo,只是不会再报错说table not found了，代码中的itemPath是从其他文档中看到的，本来是用的*itemPath=“”*,于是能从返回结果中看到很多很多信息，但是其实我们只需要用到日期，仔细看看，就把参数设置为*itemPath="rsp.photo.dates"*,这样返回值的*result*里，就只有我需要的参数了：
	
		<results>
        	<dates lastupdate="1369016824" posted="1368782260"
            taken="2013-05-17 17:18:59" takengranularity="0"/>
    	</results>    
   然后根据photoID返回拍照日期的代码为：

		USE "https://raw.github.com/yyq/yahoo/master/table.xml" AS mytable;
		select * from mytable where photo_id="8746266201" and api_key="4fb031bf5b2f138576d011ff37f31565";

#### 3.3从地理位置获取天气预报等天气信息,不多说过程了，看代码即明白 

		select * from weather.forecast where woeid in 
		(select woeid from geo.placefinder where text="39.9969808,116.3323017" and gflags="R")
	
基本过程就是，先从placefinder中，通过地理位置获取距离当前最近的woeid（估计是某个标准的位置编码吧），然后再通过woeid去weather.forecast去获得天气信息.
 
#### 3.4还有查到的用得上的photo.search的参数有：
  	
		radius="20"
		min_upload_date ="2001-01-01 00:00:00"

### 4.结果很惨淡的样子
比赛在周天的中午12点准时结束了，去西郊宾馆吃自助，那里自助真心高级，我吃了好多肉。

2点钟开始了作品展示，每个组2分钟，我们组由我展示自己的作品。比较悲剧的是，在展示到最后一个步骤，获取照片，感觉网速很慢，但是连图片都没有刷出来，主持人就说了，谢谢第29组。。。。我就只能囧囧的下台了，哎。遗憾，郁闷。

HR们和调酒师4点开始热身了，平常几乎不喝酒的我，果断的来了一杯不知道用哪些饮料调的cocktail，和队友交流着我们的失误。

颁奖结束后，确实看到了我们和别人的差距。
基本总结如下：

1. 产品简单没有关系，做好了就行（有一个获奖作品的idea就来自yahoo challenge，但是当初由于我们觉得那个简单所有没有去做）
2. 如果只是一个2分钟的demo展示，有的功能不必完全实现，有样子展示就OK
3. 需要考虑较差的网络环境，直接展示本地内容来代替网络内容
4. 我演示的时候紧张了，衰
5. 小组成员的技术很广泛，难以很好的统一，以后类似的比赛或许可以考虑大家都精通的技术，做精一个产品
6. 了解了flickr，考虑自己在iphone上弄一个，因为我发现在苹果的app store和google的play store中，中国地区的商店，都没有flickr官方的引用。很奇怪，为什么不在中国投放。然后ipad上也没有flickr。这些都可以考虑自己来做一个，反正flickr给的API看上去挺全面。

### 5.吃在hack day
作为吃货，雅虎滴伙食，那我是记忆犹新啊
哎。改天有空再谈论吃吧。我饿了，去食堂吃饭去了~


### 6.留个纪念吧
![pic1](http://farm4.staticflickr.com/3680/8750935354_98effc631d.jpg)   ![pic2](http://farm4.staticflickr.com/3759/8750605319_3f5d6bac8b.jpg)












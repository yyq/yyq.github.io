---
title: Python爬糗百热门20条并邮件分发+wxPython简易GUI+py2app转成可执行文件
author: sophie
layout: post
dsq_thread_id:
  - 4102495012
categories:
  - 计算机
  - Python
tags:
  - py2app
  - python
  - SMTP
  - wxpython
  - 爬虫
date: 2015-06-24
---
学了一阵子Python，拿来做个什么有意思的东西呢？爬糗百好了，爬到的内容，邮件分发出去。

然后又啃了两天的wxpython，做了个简易的邮件管理界面，可以在这里增加或者删除邮件，并且一键爬虫发送。

最后，索性封装成APP吧，又试了一把py2app，简单好用。

由于是让用户自行添加和删除邮箱，所以程序一定要兼顾到各种情况：比如输入的邮箱格式不合法，输入的邮箱里包含中文字符，分隔符不对，删除了全部邮箱然后又要发邮件等问题。

<!--more-->

首先是QiuBai.py：爬虫，正则匹配我们想要的内容，然后将内容稍作处理返回。

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import re
import urllib2
__author__ = 'Sophie2805'

class FetchData(object):
    def __init__(self,URL,referURL,pattern_1,pattern_2):
        self.URL = URL
        self.pattern_1 = pattern_1
        self.patttern_2 = pattern_2
        self.referURL = referURL

    def getHtml(self):
        headers = {
            'User-Agent':'Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.1.6) Gecko/20091201 Firefox/3.5.6',
            'Referer':self.referURL  #http://www.qiushibaike.com/
        }
        req = urllib2.Request(
            url = self.URL, #http://www.qiushibaike.com/text
            headers = headers)
        return urllib2.urlopen(req).read()

    def getData(self,html):
        p_1 = re.compile(self.pattern_1)
        div_content = p_1.findall(html.decode('utf8'))
        row = 0
        for m in range(len(div_content)):
            div_content[m] = re.sub(self.patttern_2,'',div_content[m])
            div_content[m] = ("---" + str(row+1) + "---" + div_content[m]).encode('utf-8')
            row += 1
        return div_content
```

接下来就是mail.py，注意，发送人的邮箱要检查下是否打开了SMTP协议。

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import smtplib
from email.mime.text import MIMEText
from email.header import Header

__author__ = 'Sophie2805'

class SMail:
    def __init__(self):
        self.sender = '***@qq.com'
        self.receiver = ['***@163.com','***@qq.com']
        self.subject = '糗百热门20条推送'
        self.username = '***@qq.com'
        self.password = '******'

    def send_mail(self,msg):
        try:
            self.msg = MIMEText(msg,'plain','utf-8')
            self.msg['Subject'] = Header(self.subject,'utf-8')
            self.msg['To'] = ','.join(self.receiver)
            self.msg['From'] = self.sender
            self.smtp = smtplib.SMTP()
            self.smtp.connect('smtp.qq.com')
            self.smtp.login(self.username,self.password)
            self.smtp.sendmail(self.msg['From'],self.receiver,self.msg.as_string())
            self.smtp.quit()
            return '1'
        except Exception, e:
            return str(e)</pre>
```

最后，用wxpython做了个简易的收件人管理界面，Spider_QiuBai.py，实现：增加邮件、删除邮件、一键爬虫发送糗百热门20条到邮件列表。

当然，为了程序正确运行，会有各种验证，比如输入的邮箱格式是否合法，发送时检查receiver列表是否为空等。

![x](2015-06-24-1.png)

&nbsp;

程序比较简单，代码如下：

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
__author__ = 'Sophie2805'

import wx
import wx.grid
from mail import *
from QiuBai import *

class Frame(wx.Frame):
    QiuBai = None
    content = None
    m = SMail()
    def __init__(self, parent=None, title='Get the Hottest QiuBai!',size=(450, 630),
                 style=wx.DEFAULT_FRAME_STYLE ^ (wx.RESIZE_BORDER |wx.MAXIMIZE_BOX)):
        wx.Frame.__init__(self, parent=parent,  title=title, size=size,style = style)
        self.Centre()
        panel = wx.Panel(self,-1)

        self.text_Email = wx.StaticText(panel,1, "You want it too? Add your email below. Using ';' between multiple emails. Existing emails will not be added again.")
        self.input_Email = wx.TextCtrl(panel,2)
        self.button_Email = wx.Button(panel,3,label='+')

        '''Bind with function'''
        self.button_Email.Bind(wx.EVT_BUTTON,self.add)

        self.text_DeleteEmail = wx.StaticText(panel,4, "Existing receivers are listed below. You can delete by double clicking them and clicking on '-' button. Please note that it can hold 20 emails at most one time.")

        '''grid'''
        self.grid = wx.grid.Grid(panel,5)
        self.grid.CreateGrid(20,1)
        self.grid.EnableEditing(False)
        self.grid.SetColSize(0,200)
        self.grid.SetColLabelValue(0,'Emails')
        for i in range(len(self.m.receiver)):
                self.grid.SetCellBackgroundColour(i,0,"grey")
                self.grid.SetCellValue(i,0,self.m.receiver[i])

        self.grid.Bind(wx.grid.EVT_GRID_CELL_LEFT_DCLICK,self.grid_change_color)

        self.button_DeleteEmail = wx.Button(panel,6,label='-')
        self.button_Send = wx.Button(panel,7,label='send')

        '''Bind with function'''
        self.button_Send.Bind(wx.EVT_BUTTON,self.sendmail)
        self.button_DeleteEmail.Bind(wx.EVT_BUTTON,self.d_emails)

        '''BoxSizer'''
        '''================================================================'''
        add_Email = wx.BoxSizer() #input box and add button
        add_Email.Add(self.input_Email,proportion=4,flag = wx.LEFT, border = 5)
        add_Email.Add(self.button_Email,proportion=1,flag=wx.LEFT,border=15)

        ae = wx.BoxSizer(wx.VERTICAL)
        ae.Add(self.text_Email,proportion =4, flag=wx.ALL, border=5)
        ae.Add(add_Email,proportion=2, flag=wx.LEFT|wx.BOTTOM|wx.RIGHT, border=5)

        h_b = wx.BoxSizer()#send button and delete button
        h_b.Add(self.button_Send,flag=wx.LEFT,border=5)
        h_b.Add(self.button_DeleteEmail,flag=wx.LEFT,border=250)

        de = wx.BoxSizer(wx.VERTICAL)
        de.Add(self.text_DeleteEmail,proportion=2,flag=wx.ALL, border=5)
        de.Add(self.grid,proportion = 4,flag=wx.LEFT, border=80)
        de.Add(h_b,proportion=1,flag=wx.TOP, border=10)

        vBoxSizer = wx.BoxSizer(wx.VERTICAL)
        vBoxSizer.Add(ae,proportion=2,flag=wx.ALL,border=5)
        vBoxSizer.Add(de,proportion=4,flag=wx.LEFT|wx.BOTTOM|wx.RIGHT,border=5)
        panel.SetSizer(vBoxSizer)

    #delete the emails that cell bk color changed to yellow
    def d_emails(self,evt):
        count_yellow = 0
        for i in range(len(self.m.receiver)):
            if self.grid.GetCellBackgroundColour(i,0)==(255, 255, 0, 255): # yellow
                count_yellow += 1

        if count_yellow == 0:
            wx.MessageBox('Wake up!\nSelect then delete!')

        else:
            l = len(self.m.receiver)
            self.m.receiver=[]
            for i in range(l):
                if self.grid.GetCellBackgroundColour(i,0)==(128, 128, 128, 255):#grey
                    self.m.receiver.append(self.grid.GetCellValue(i,0))
                self.grid.SetCellValue(i,0,'')#clear the cell's value
                self.grid.SetCellBackgroundColour(i,0,'white')

            for i in range(len(self.m.receiver)):
                self.grid.SetCellBackgroundColour(i,0,"grey")
                self.grid.SetCellValue(i,0,self.m.receiver[i])
            self.grid.ForceRefresh()

    #change cell bk color when double click the cell. The cell without value will not be changed
    def grid_change_color(self,evt):
        if self.grid.GetCellValue(self.grid.GridCursorRow,self.grid.GridCursorCol) != '':
            if self.grid.GetCellBackgroundColour(self.grid.GridCursorRow,self.grid.GridCursorCol)==(128, 128, 128, 255):#grey
                self.grid.SetCellBackgroundColour(self.grid.GridCursorRow,self.grid.GridCursorCol,'yellow')
            else:
                self.grid.SetCellBackgroundColour(self.grid.GridCursorRow,self.grid.GridCursorCol,'grey')
        self.grid.ForceRefresh()

    #add the emails that user input
    def add(self,evt):
        x = str(self.input_Email.GetValue().encode('utf-8'))
        if len(x) == 0:
            wx.MessageBox('Hey dude, input something first!')
        elif len(x) != len(x.decode('utf-8')):#non ASCII character contained
            wx.MessageBox('Hey dude, please input English character only!')
        else:
            x = x.split(';')
            tag = 0
            for i in range(len(x)):
                if len(x[i]) != 0:
                    if x[i] in self.m.receiver:
                        x[i]=''#existing emails, set to ''
                    elif self.validate_email(x[i]) == None:
                        wx.MessageBox('Hey dude, eyes wide open! \nPlease input like this: xxx@xxx.com;eee@eee.org')
                        tag = 1
                        break
            if tag == 0: # all emails are valid
                for item in x:
                    if len(item) != 0:
                        self.m.receiver.append(item)
                for i in range(len(self.m.receiver)):
                    self.grid.SetCellBackgroundColour(i,0,"grey")
                    self.grid.SetCellValue(i,0,self.m.receiver[i])
                self.input_Email.Clear()
                self.grid.ForceRefresh()

    #validate the emails user input
    def validate_email(self,s):
        pattern = "^[a-zA-Z0-9\._-]+@([a-zA-Z0-9_-]+\.)+([a-zA-Z]{2,3})$"
        p = re.compile(pattern)
        return p.match(s)

    def sendmail(self,evt):
        if len(self.m.receiver) == 0:
            wx.MessageBox('Are you kidding me?\n Add some receivers then send!')
        else:
            self.QiuBai = FetchData('http://www.qiushibaike.com/text','http://www.qiushibaike.com/',
                                '&lt;div class="content"&gt;[\n\s]+.+[\n\s]','[&lt;div class="content"&gt;&lt;br/&gt;\n]')
            self.content = self.QiuBai.getData(self.QiuBai.getHtml())
            msg = '\n\n'.join(self.content)
            result = self.m.send_mail(msg)
            if result == '1':
                wx.MessageBox('The hottest QiuBai had been sent.\n Enjoy it ^_^ !')
            else:
                wx.MessageBox('Some exception occurred :-(\nTry again later...\n'+result)
class GetQiuBaiApp(wx.App):
    def OnInit(self):
        frame = Frame()
        frame.Show()
        return True

if __name__ == "__main__":
    app = GetQiuBaiApp()
    app.MainLoop()

```
最后就是Python转APP了：

（1）在MAC终端里输入pip install py2app安装py2app

（2）创建setup.py，我的project位于/Users/Sophie/PycharmProjects/QiuBai，所以cd到这个路径下，然后vi setup.py，在这个文件里输入如下内容（提前准备个icns图标，放到project路径下）

```
py2app build script for MyApplication

Usage:

python setup.py py2app

from setuptools import setup

OPTIONS = {

&#8216;iconfile&#8217;:&#8217;lemon.icns&#8217;

}

setup(

app=[&#8220;Spider_QiuBai.py&#8221;],

options={&#8216;py2app&#8217;: OPTIONS},

setup_requires=[&#8220;py2app&#8221;],

)
```

（3）rm -rf build dist（如果重复多次打包的话，这个命令是需要的，第一次不需要）

（4）python setup.py py2app 用这个命令打包，打包好的app会存放在该路径下的dist路径下，我的如下：/Users/Sophie/PycharmProjects/QiuBai/dist

![x](2015-06-24-2.png)

至此，这个小玩意儿已经做好了，其中略麻烦的可能就是正则还有界面了 Hope you enjoy it ^ ^

&nbsp;

 [1]: http://yyqing.me/wp-content/uploads/2015/07/1.png
 [2]: http://yyqing.me/wp-content/uploads/2015/07/2.png
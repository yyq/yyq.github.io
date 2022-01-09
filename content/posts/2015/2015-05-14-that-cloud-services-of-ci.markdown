---
title: That Cloud Services of CI
author: young
layout: post
dsq_thread_id:
  - 3762034113
categories:
  - 计算机
  - Infrastructure
tags:
  - build
  - CI
  - cloud
  - coverage
  - coveralls
  - python
  - service
  - travis
format: aside
date: 2015-05-14
---
When we find some open source projects on github( e.g. <a href="https://github.com/cucumber/cucumber" target="_blank">ruby-cucumber</a>). We can found some green icon to display the status of the project on the README.md file. Like these:

![x](2015-05-14-10.18.58.png)

, it looks very attractive to me. Especially the build passing status and the code coverage. Even I&#8217;m not working for the CI in our company, I&#8217;m still interested in it.

When push some code to my code base, then automatically building the code, running tests, and showing the build status & code coverage real-time on the readme page, That&#8217;s COOL! When I look into more, the [travis-ci.org][2] and the [coveralls.io][3] are free for open source project on github.

Let me have a try.

Since recently I&#8217;m learning python. First, I created a [helloworldpython][4] project on my github.

# The Travis-CI

it supports most popular languages. After signing in it with github account, open the service for my helloworld project. After the authentication for travis-ci, it will add an service for the project, we can found that on the setting of the project:

<!--more-->
![x](2015-05-14-09.31.01.png)

##  simple config of travis-ci python project

Then, we should add the config file to this project, it means a .travis.yml file on the root folder in the project. As for the format of a python project, refer to the help page [here][6], the first time I add a very simple config file for it:

```
language: python
python:
  - "3.3"
  - "2.7"
  - "2.6"
install: python ClassX.py
script: python ClassX.py</pre>
```

Four parts:

  * we are using python language for this project
  * which versions of python we want to our project to compatible. I just add the 3.3, 2.7 and 2.6 there.
  * the install part describes the command the install dependencies.
  * the script part will include the command to run this project, you&#8217;d better run some tests here to make your project runnable.

my projects are very simple , so i just using python file.py on the install part and the script part. The part 2 can makes me very confident if my code can pass all the builds in different versions, after the three versions configured:

![x](2015-05-14-09.49.13.png)

## which commands it executed for me

The travis-ci will using different version python to build this project, we can see that failed when in python 3.3. So what&#8217;s going on there. Click the 3.1 item, we can get more details as the following:

![x](2015-05-14-09.54.29.png)

we can found these following interesting things:

  * It&#8217;s using docker container to do this build.
  * Then it will clone the project with 50-depth commits. The branch name can be specified in the travis-ci webpage. I&#8217;m using master now.
  * It will install python 3.3 in the container and confirm the version of python and pip.
  * the last process is running the command I wrote in the .travis.yml file. And we can see why this failed.
  * For each step, we can see the time costing on the right.
  * Pay attention to the line 3, if we stretch it, we can see more details info of the system. This will help you to troubleshoot when somethings happens.

# <span style="line-height: 1.625;">The <a href="https://coveralls.io">Coveralls.io</a></span>

<span style="line-height: 1.625;">So, this tool is used to get code coverage info. Also easily login with github id, also free for open source project on github.</span>

<span style="line-height: 1.625;">For my python project, it&#8217;s easy to find help doc <a href="https://coveralls.zendesk.com/hc/en-us/articles/201342869-Python">here</a>. </span>

Basically, before running the project , three steps to got code coverage info.

First, install the python package &#8220;coverage&#8221; and add this action into the .travis.yml file&#8217;s &#8220;install&#8221; part.

Second, after build the project, using coverage to run some code or tests.

Third, in the &#8220;after_success&#8221; part of .travis.yml file, using coveralls. Done.

So, my result .travis.yml file looks like this:

<pre class="lang:yaml decode:true ">
language: python
python:
  - "2.7"
  - "2.6"
install: 
  - python ClassX.py
  - pip install python-coveralls
script: 
  - coverage run ClassX.py
  - coverage report -m
after_success:
  - coveralls
</pre>

the line coverage report -m, after my simple use the coverage, I found the coverage report can be useful. Until now, I can see the basic coverage info in from the travis-ci&#8217;s console output like this:

![x](2015-05-19-18.36.45.png)

And if we want to know which line are covered or not. Details in the [coveralls.io][3] looks like:

![x](2015-05-19-18.42.20.png)
clear. Right, my next question is the number on the right of every line. The 2 means this line are executed twice ?

&nbsp;

 [1]: http://yyqing.me/wp-content/uploads/2015/05/屏幕快照-2015-05-14-10.18.58.png
 [2]: https://travis-ci.org
 [3]: https://coveralls.io
 [4]: https://github.com/yyq/HelloWorldPython
 [5]: http://yyqing.me/wp-content/uploads/2015/05/屏幕快照-2015-05-14-09.31.01.png
 [6]: http://docs.travis-ci.com/user/languages/python/
 [7]: http://yyqing.me/wp-content/uploads/2015/05/屏幕快照-2015-05-14-09.49.13.png
 [8]: http://yyqing.me/wp-content/uploads/2015/05/屏幕快照-2015-05-14-09.54.29.png
 [9]: http://yyqing.me/wp-content/uploads/2015/05/屏幕快照-2015-05-19-18.36.45.png
 [10]: http://yyqing.me/wp-content/uploads/2015/05/屏幕快照-2015-05-19-18.42.20.png
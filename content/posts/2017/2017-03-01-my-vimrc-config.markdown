---
Title: 我的vim配置说明
author: Young
date: 2017-03-01
tags: [vim, vimrc, bundles, plugin]
Status: public
---
参考了不少书籍和网站，最终形成一个自己觉得顺手的vim配置文件，存在github的gist了，有兴趣的朋友可以看看，交流交流。
[.vimrc](https://gist.github.com/yyq/b701e781b00822fb41f271eef42c9767)
[.vimrc.bundles](https://gist.github.com/yyq/750d1a1b23ea5050a7f4cf34d98b0864)

要相关插件都正常工作的话，需要手动执行的一些操作如下：
 
 1. mkdir ~/.vim
 2. touch .vimrc
 3. touch .vimrc.bundles
 4. fill the file content(vimrc, vimrc.bundles)
 5. Download Vundle for VIM ```git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim```
 6. open vim, and run ```:BundleInstall```

 The command to install ctags on my mac:
 ```brew install ctags```
 
 The command to install YouCompleteMe
 ```
 cd ~/.vim/bundle/YouCompleteMe
 ./install.py --clang-completer
 ```
有的插件对VIM有版本要求，版本号不能太低了。我是用的macOS，安装了最近的8.0版本，绝大部分插件都能正常工作
 
关于各个插件的详细使用说明和配置说明的话，可以参考vimrc.bundles里面的各个github链接，看他们的readme，或者直接看他们源代码，代码都存在~/.vim/bundle里面了

我把我的这两个配置文件也贴在这里好了

<script src="https://gist.github.com/yyq/b701e781b00822fb41f271eef42c9767.js"></script>

<script src="https://gist.github.com/yyq/750d1a1b23ea5050a7f4cf34d98b0864.js"></script>




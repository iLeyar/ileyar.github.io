---

title: Ubuntu 上 vim 使用及其配置
tag:
- vim
- ubuntu
categories: vim

---


>前言：自从换了 Ubuntu，Sublime-text3 不能完美的解决中文输入法的问题一直困扰着我。系统自带的文本编辑器功能又太单一,无法满足我的需求。偶然间发现 vim 的强大，并且也支持添加例如 emmet，markdown 之类的插件，于是决定好好开始折腾这个传说中的神器。
>


## vim 快捷键

使用 vim 自带的 Vim 教程就基本满足日常使用了，我是每天跟着教程都动手操作几遍，慢慢锻炼肌肉记忆。现在也能顺畅的进行进行文本编辑了。

方法：在终端输入如下命令即可开启训练之旅啦。

```
vimtutor
```
<!--more-->
## 插件

这里列出一些我自己使用的插件及配置。

- [Vundle.vim](https://github.com/gmarik/Vundle.vim):vim 扩展管理器，类似于 sublime-text 的 package control, 安装方法略。:vim 扩展管理器，类似于 sublime-text 的 package control, 安装方法自行移步作者 github。

    这里记录下使用方法：

   vim 中操作：
 
    ```bash
    :PluginList        # 列出配置的插件
    :PluginInstall     # 安装插件;后面添加 "!" 则等同于 :PluginUpdate 
    :PluginUpdate      # 更新插件
    :PluginSearch xx   # 搜索插件, xx 为搜索关键词；后面添加 "!" 则可刷新本地缓存
    :PluginClean       # 卸载插件;后面添加 "!" 则直接卸载。 
    :h vundle          # 查看帮助

    ```
   
  Terminal 中操作：
    
    ```bash
    vim +PluginInstall +qall

    ```

- [emmet-vim](https://github.com/mattn/emmet-vim/):这是我在 sublime-text 3 中必备的神器。具体也不作介绍，Google 就可以搜到一大堆实用教程。

    这里留下某个参考教程：[Emmet.vim 教程](http://www.zfanw.com/blog/zencoding-vim-tutorial-chinese.html)

    vim 中查看帮助：

    ```bash
    :help emmet

    ```

- [vim-markdown](https://github.com/plasticboy/vim-markdown):支持语法高亮，批量修改 header 大小等，更多功能还在深入研究中..

- [Goyo.vim](https://github.com/junegunn/goyo.vim):与下两个插件一起搭配，即可进入优雅的写作模式。

    ```
    - :Goyo              # 开启写作模式，也可使用 F12 快捷键进入
    - :Goyo [width]      # 开启并设置宽度，默认宽度为 80
    - :Goyo!             # 关闭写作模式，也可以使用 :q[uit], :clo[se], :tabc[lose], 或者 :Goyo

    ```

- [seoul256.vim](https://github.com/junegunn/seoul256.vim):一个优雅漂亮的配色。
- [limelight.vim](https://github.com/junegunn/limelight.vim):在写作模式中开启，调节背景字体暗度。

    ```
    :Limelight[0.0~1.0]        # 开启并设置暗度
    :Limelight!                # 关闭
    :Limelight!![0.0~1.0]      # 开关
    
    ```
    某些配色如果没有生效，则加入如下代码：

    ```
    g:limelight_conceal_ctermfg   或者 
    g:limelight_conceal_guifg

    ```

- [Syntastic](https://github.com/scrooloose/syntastic):语法检查 

   
## 配置

这里放一份我的个人配置，配置文件为 ~/.vimrc

    ```bash

filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

Plugin 'gmarik/Vundle.vim'
Plugin 'tpope/vim-fugitive'
Plugin 'mattn/emmet-vim'
Plugin 'L9'
Plugin 'git://git.wincent.com/command-t.git'
Plugin 'file:///home/gmarik/path/to/plugin'
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
Plugin 'user/L9', {'name': 'newL9'}
Plugin 'godlygeek/tabular'
Plugin 'plasticboy/vim-markdown'
Plugin 'junegunn/goyo.vim'
Plugin 'junegunn/seoul256.vim'
Plugin 'junegunn/limelight.vim'
Plugin 'jdevera/vim-cs-explorer'
Plugin 'Syntastic'

call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on

" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal

" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line

let g:vim_markdown_folding_disabled = 1
autocmd BufNewFile,BufReadPost *.md set filetype=markdown
"Used as $x^2$, $$x^2$$, escapable as \$x\$ and \$\$x\$\$.
let g:vim_markdown_math = 1
let g:vim_markdown_frontmatter = 1


"nnoremap <Leader>w :Goyo<CR>
map <silent> <F12> :Goyo<CR>
autocmd User GoyoEnter Limelight
autocmd User GoyoLeave Limelight!
let g:goyo_width = 90
let g:goyo_margin_top = 4
let g:goyo_margin_bottom = 4
let g:goyo_linenr = 0

" Color name (:help cterm-colors) or ANSI code
let g:limelight_conceal_ctermfg = 'gray'
let g:limelight_conceal_ctermfg = 240

" Color name (:help gui-colors) or RGB color
let g:limelight_conceal_guifg = 'DarkGray'
let g:limelight_conceal_guifg = '#777777'

" Default: 0.5
let g:limelight_default_coefficient = 0.7

map <leader>d :ColorSchemeExplorer<CR>

" seoul256 (dark):
" Range:   233 (darkest) ~ 239 (lightest)
" Default: 237
colo seoul256
let g:seol256_background = 233

" emmet:
let g:user_emmet_install_global = 0
autocmd FileType html,css EmmetInstall
let g:user_emmet_expandabbr_key = '<Tab>'

set backspace=indent,eol,start
set autoindent
set nu!
syntax enable
syntax on
set ai!
set showmatch
set incsearch
set ruler
set smartindent
set laststatus=2
set statusline=\ %<%F[%1*%M%*%n%R%H]%=\ %y\ %0(%{&fileformat}\ %{&encoding}\ %c:%l/%L%)
set cursorline 
set foldmethod=indent
set mouse=v
set showcmd
set hlsearch
set nocompatible
set fileencodings=utf-8,gb18030,gbk,big5

" Syntastic
set statusline+=%#warningmsg#
set statusline+=%{SyntasticStatuslineFlag()}
set statusline+=%*

let g:syntastic_always_populate_loc_list = 1
let g:syntastic_auto_loc_list = 1
let g:syntastic_check_on_open = 1
let g:syntastic_check_on_wq = 0



    ```
   
 
<hr>

## 小结：

关于 vim 还有很多需要慢慢研究的和学习的地方，边用边研究吧。Enjoy~





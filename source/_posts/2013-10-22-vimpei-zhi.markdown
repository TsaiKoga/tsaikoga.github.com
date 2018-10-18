---
layout: post
title: "Vim配置"
date: 2013-10-22 14:24
comments: true
categories: [Editor]
---

说到vim，大家应该都不陌生。它是类似于vi的编辑器，在vi的基础上增加了很多新特性。

![图片无法显示](/images/posts/2013-10-22/vim.jpg "Vim图")

## 目录
#### [1 vim 中容易忘掉但非常有用的命令](#1)
#### [2 .vimrc配置文件](#2)
##### [2.1 基本配置说明](#2.1)
##### [2.2 括号补全](#2.2)
#### [3 VIM 常用插件](#3)
##### [3.1 NERDTree 目录树](#3.1)
##### [3.2 NERDCommenter 注释](#3.2)
##### [3.3 Scsope 跳转](#3.3)

<br/>
<br/>

<h2 id='1'>1 vim 中容易忘掉但非常有用的命令</h2>

-------------------------

- 多选        ctrl+v(mac), ctrl+q(windows/linux)
- 打开文件    :open filename
- 注释        NERD_commenter命令是\\cc或,cc
- 解除注释     NERD_commenter命令是\\cu或,cu
- 开启高亮    :syntax on
- 开启行号    :set number
- 跳转方法    cscope 的命令 ctl+]（必须先建立数据库和连接）
- 回退方法    cscope 的命令 ctl+t
- 跳到目录树  ctrl+w+h
- 跳回文件页  ctrl+w+l


<h2 id='2'>2 .vimrc配置文件</h2>

------------------------------

<h3 id="2.1">2.1 基本配置说明</h3>

以下内容复制到用户目录下的.vimrc文件，可以更改vim配置，使Vim更美观，更好用:

``` sh
    set nocompatible            " 关闭 vi 兼容模式
		syntax on                   " 自动语法高亮
		colorscheme molokai         " 设定配色方案
		set number                  " 显示行号
		set cursorline              " 突出显示当前行
		set ruler                   " 打开状态栏标尺
		set shiftwidth=4            " 设定 << 和 >> 命令移动时的宽度为 4
		set softtabstop=4           " 使得按退格键时可以一次删掉 4 个空格
		set tabstop=4               " 设定 tab 长度为 4
		set nobackup                " 覆盖文件时不备份
		set autochdir               " 自动切换当前目录为当前文件所在的目录
		filetype plugin indent on   " 开启插件
		set backupcopy=yes          " 设置备份时的行为为覆盖
		set ignorecase smartcase    " 搜索时忽略大小写，但在有一个或以上大写字母时仍保持对大小写敏感
		set nowrapscan              " 禁止在搜索到文件两端时重新搜索
		set incsearch               " 输入搜索内容时就显示搜索结果
		set hlsearch                " 搜索时高亮显示被找到的文本
		set noerrorbells            " 关闭错误信息响铃
		set novisualbell            " 关闭使用可视响铃代替呼叫
		set t_vb=                   " 置空错误铃声的终端代码
		" set showmatch               " 插入括号时，短暂地跳转到匹配的对应括号
		" set matchtime=2             " 短暂跳转到匹配括号的时间
		set magic                   " 设置魔术
		set hidden                  " 允许在有未保存的修改时切换缓冲区，此时的修改由 vim 负责保存
		set guioptions-=T           " 隐藏工具栏  （等号前后不能空格）
		set guioptions-=m           " 隐藏菜单栏
		set smartindent             " 开启新行时使用智能自动缩进
		set backspace=indent,eol,start
																" 不设定在插入状态无法用退格键和 Delete 键删除回车符
		set cmdheight=1             " 设定命令行的行数为 1
		set laststatus=2            " 显示状态栏 (默认值为 1, 无法显示状态栏)
		set statusline=\ %<%F[%1*%M%*%n%R%H]%=\ %y\ %0(%{&fileformat}\ %{&encoding}\ %c:%l/%L%)\
																" 设置在状态行显示的信息
		set foldenable              " 开始折叠
		set foldmethod=syntax       " 设置语法折叠

		"窗口分割时,进行切换的按键热键需要连接两次,比如从下方窗口移动
		"光标到上方窗口,需要<c-w><c-w>k,非常麻烦,现在重映射为<c-k>,切换的
		"时候会变得非常方便.
		nnoremap <C-h> <C-w>h
		nnoremap <C-j> <C-w>j
		nnoremap <C-k> <C-w>k
		nnoremap <C-l> <C-w>l
```

<br/>

<h3 id='2.2'>2.2 括号补全：</h3>

``` sh
		inoremap (	()<Esc>i				" inoremap表示输入模式下的匹配,当有"("时，匹配")"并退出重新进入"i编辑模式"
		inoremap [	[]<Esc>i
		inoremap {	{}<Esc>i
		autocmd Syntax html,vim inoremap < <lt>><Esc>i | inoremap > <c-r>=ClosePair('>')<CR>
		inoremap ) <c-r>=ClosePair(')')<CR>
		inoremap ] <c-r>=ClosePair(']')<CR>
		inoremap } <c-r>=CloseBracket()<CR>
		inoremap " <c-r>=QuoteDelim('"')<CR>
		inoremap ' <c-r>=QuoteDelim("'")<CR>

		function ClosePair(char)
			if getline('.')[col('.') - 1] == a:char
				return "\<Right>"
			else
				return a:char
			endif
		endf

		function CloseBracket()
			if match(getline('.' + 1), '\s*}') < 0
				return "\<CR>}"
			else
				return "\<Esc>j0f}a"
			endif
		endf

		function QuoteDelim(char)
			let line = getline('.')
			let col = col('.')
			if line[col-2] == "\\"
				"Inserting a quoted quotation mark into string
				return a:char
			elseif line[col-1] == a:char
				"Escaping out of the string
				return "\<Right>"
			else
				"Starting a String
				return a:char.a:char."\<Esc>i"
			endif
		endf
```

<br/>
<br/>


<h2 id='3'>3 VIM 常用插件</h2>

--------------------------

<h3 id='3.1'>3.1 NERDTree 目录树</h3>

功能：用于展示目录树
安装：下载之后，在 .vim/ 目录中创建 plugin/ 和 doc/ 目录，
     将 NERDTree.txt 放入 doc/，将 NERDTree.vim 放入 plugin/。
用法：https://www.jianshu.com/p/eXMxGx

``` sh
		"-----------------------------------------------------------------
		" plugin - NERD_tree.vim 以树状方式浏览系统中的文件和目录
		" :ERDtree 打开NERD_tree         :NERDtreeClose    关闭NERD_tree
		" o 打开关闭文件或者目录         t 在标签页中打开
		" T 在后台标签页中打开           ! 执行此文件
		" p 到上层目录                   P 到根目录
		" K 到第一个节点                 J 到最后一个节点
		" u 打开上层目录                 m 显示文件系统菜单（添加、删除、移动操作）
		" r 递归刷新当前目录             R 递归刷新当前根目录
		"-----------------------------------------------------------------
		" F3 NERDTree 切换
		map <F3> :NERDTreeToggle<CR>
		imap <F3> <ESC>:NERDTreeToggle<CR>
```

<br/>

<h3 id='3.2'>3.2 NERDCommenter 注释</h3>

功能：用于快速注释和解除注释，使用命令请查看顶部

``` sh
		"-----------------------------------------------------------------
		" plugin - NERD_commenter.vim   注释代码用的，
		" [count],cc 光标以下count行逐行添加注释(7,cc)
		" [count],cu 光标以下count行逐行取消注释(7,cu)
		" [count],cm 光标以下count行尝试添加块注释(7,cm)
		" ,cA 在行尾插入 /* */,并且进入插入模式。 这个命令方便写注释。
		" 注：count参数可选，无则默认为选中行或当前行
		"-----------------------------------------------------------------
		let NERDSpaceDelims=1       " 让注释符与语句之间留一个空格
		let NERDCompactSexyComs=1   " 多行注释时样子更好看
```

<br/>

<h3 id='3.3'>3.3 Scsope 跳转</h3>

cscope 可以说是 ctags 的升级版本
功能：可以对函数以及部分类型定义进行跳转

``` sh
sudo apt-get install cscope
```

使用举例：
在终端下，转到你源码的所在目录然后

``` sh
cscope -Rbkq  <回车>
```

##### 参数说明：
> R 表示把所有子目录里的文件也建立索引
>
> b 表示cscope不启动自带的用户界面，而仅仅建立符号数据库
>
> q 生成cscope.in.out和cscope.po.out文件，加快cscope的索引速度
>
> k 在生成索引文件时，不搜索/usr/include目录

之后会在当前目录生成几个文件，  cscope.in.out和cscope.po.out文件,cscope.out，

vim的normal模式下输入[建立cscope数据库]

``` sh
:cs add cscope.out
```

或者在 .vimrc 中加入，这样每次打开vim就可以直接使用cscope了。

``` sh
if filereadable("cscope.out")
    cs add cscope.out
endif
```

默认情况下cscope值会在当前目录下针对c、iex和yacc（扩展名分别为.c、.h、.I、.y）程序文件进行解析（如果指定了-R参数则包含其自身的子目录）。
这样出现的问题就是，我们对于C++、Java、PHP 等文件怎么办，
解决方案是：我们可以生成一个名为cscope.finds的文件列表，并交由cscope去解析。
在Linux系统中，生成这个文件列表的方法是：

``` sh
find –name "*.php" > cscope.files
```

然后运行下面命令 **重新生成数据库** 就OK了。

``` sh
cscope –b
```

然后进入文件，输入你需要查找的方法，就会直接跳转到该方法：

``` sh
:cs find g checkStoreEmployee
```

也可以在 .vimrc 中加入：

``` sh
if has("cscope")

  set csprg=/usr/local/bin/cscope

  set csto=0

  set cst

  set nocsverb

  " add any database in current directory

  if filereadable("cscope.out")

    cs add cscope.out

    " else add database pointed to by environment

  elseif $CSCOPE_DB != ""

    cs add $CSCOPE_DB

  endif

  set csverb

endif
```

在 vim normal 模式下执行：

```sh
:source ~/.vimrc
```

即可方便使用 ```ctl + ]```来快速跳到方法 和 ```ctl + t``` 回退到原方法。

<br/>



_感谢使用vim配置，记住要将vim编辑器重启后才生效。_

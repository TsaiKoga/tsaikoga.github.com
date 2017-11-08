---
layout: post
title: "Vim配置"
date: 2013-10-22 14:24
comments: true
categories: [Editor]
---

说到vim，大家应该都不陌生。它是类似于vi的编辑器，在vi的基础上增加了很多新特性。

![图片无法显示](/images/posts/2013-10-22/vim.jpg "Vim图")

#### vim 中容易忘掉但非常有用的命令

- 多选        ctrl+v(mac), ctrl+q(windows/linux)
- 打开文件    :open filename
- 注释        NERD_commenter命令是\\cc或,cc
- 解除注释     NERD_commenter命令是\\cu或,cu
- 开启高亮    :syntax on
- 开启行号    :set number

####.vimrc配置文件

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

接下来是括号补全：

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

下列两款vim插件非常好用，安装完，也必须在.vimrc中添加如下内容：

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

_感谢使用vim配置，记住要将vim编辑器重启后才生效。_

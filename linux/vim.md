# Vim

我的一些Vim设置

/root/.vimrc

```
set nu
set autoread
set autoindent
set smartindent
set hlsearch
set incsearch
set ruler
set showmatch
set showmode
set showcmd
set cuc
set cul
colorscheme desert
syntax on
"使用F9进入粘贴模式，防止格式乱（自动缩进
set pastetoggle=<F9>

filetype plugin indent on 

"设置标题
autocmd BufNewFile *.sh exec ":call SetTitle()"
func SetTitle()
  if expand("%:e") == 'sh'
    call setline(1,"#*************************************************************")
    call setline(2,"#  Author	:  Neal Zhao")
    call setline(3,"#  Date		:  ".strftime("%c"))
    call setline(4,"#  FileName	:  " .expand("%"))
    call setline(5,"#*************************************************************")
    call setline(6,"#!/bin/bash")
    call setline(7,"")
    call setline(8,"")
  endif
endfunc
"定位到末行
autocmd BufNewFile * normal G

"自动补全
:inoremap ( ()<ESC>i
:inoremap ) <c-r>=ClosePair(')')<CR>
:inoremap { {<CR>}<ESC>O  
:inoremap } <c-r>=ClosePair('}')<CR>
:inoremap [ []<ESC>i
:inoremap ] <c-r>=ClosePair(']')<CR>
:inoremap < <><ESC>i
:inoremap > <c-r>=ClosePair('>')<CR>
:inoremap " ""<ESC>i
:inoremap ' ''<ESC>i
function! ClosePair(char)
  if getline('.')[col('.') - 1] == a:char
    return "\<Right>"
  else
    return a:char
  endif
endfunction
"打开文件类型检测, 加了这句才可以用智能补全
set completeopt=longest,menu
```




---
layout: post
title: 개발에 유용한 툴들
author: 하얀눈길
description: null
tags: [developer-tool,개발툴]
featuredImage: null
img: null
categories: tool
date: '2018-05-18'
extensions:
  preset: gfm

---


>가끔 장비를 교체하는 경우 전에 설치했던 툴들이 기억이 나지 않을 때가 있습니다. 세월이 갈수록 점점.. 더..
>그래서 내가 사용하는 툴 목록을 정리해 두려 합니다.


## DB Client

**MYSQL**
 - MySQL WorkBench - https://www.mysql.com/products/workbench/ :-1:
 - HeidiSQL - https://www.heidisql.com/ :+1:

**Vertica**
 - DBeaver - https://dbeaver.io/


## IDE & Editor

**programming IDE**
 - jet brains All Products pack - https://www.jetbrains.com/store/?fromMenu#edition=commercial
 -- including Intellij, ReSharper and other IDEs  
 -- intelliJ, PhpStorm정도 쓴다면 그냥 all pack을 사는 것도.. :smirk:

**Markdown Editor**
 - stackedit.io - web editor, https://stackedit.io
 - markdown emoji code - https://gist.github.com/rxaviers/7360908 :+1:

**Text Editor**
 - sublime text 3 - https://www.sublimetext.com/ :+1:

## Terminal
 - iterm - https://www.iterm2.com/
 - putty - https://www.putty.org/

## Performance Test
- nGrinder : http://naver.github.io/ngrinder/

## ETC Tools

 - calculator - https://numi.io/ only mac :+1:
 - android emulator - https://kr.bignox.com/ :+1:
 - screen capture -  
https://evernote.com/intl/ko/products/skitch
 - tcp capture - https://www.wireshark.org/
 - SCM - https://github.com/
 - countdown - https://itunes.apple.com/kr/app/countdown-widget/id506996014?mt=12
 - rest api - https://www.getpostman.com/
 - file transfer(any platform) - https://send-anywhere.com/

## ETC Configure

 - vim : https://github.com/amix/vimrc

```bash
" syntax Highlighting
if has("syntax")
    syntax on
endif

colorscheme evening

" set cindent
set nu
set ts=4

set nocp                    " 'compatible' is not set
filetype plugin on          " plugins are enabled

let g:netrw_banner = 0
let g:netrw_liststyle = 3
let g:netrw_browse_split = 4
let g:netrw_altv = 1
let g:netrw_winsize = 25

augroup ProjectDrawer
  autocmd!
  autocmd VimEnter * :Vexplore
augroup END
```




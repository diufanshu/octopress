---
layout: post
title: "vim-latex 整合latexmk"
date: 2016-02-05 00:40:39 +0800
comments: true
categories: [latex] 
---
今日体验了一把vim-latex 插件，感觉很棒，然而挑剔的我还是遇到了几个问题：

1. OSX 系统中Alt 键相关的快捷键失效。

2. 插件自带的编译快捷键```\ll```只能用pdflatex 引擎，并且有编号的时候需要多编译几次，而我通常使用的是xetex 引擎。

<!--more-->

问题1 比较容易解决，把用到的Alt 相关快捷键重新绑定到Meta(Option)键上，经过一番Google，参考vim-latex 插件文档[^1] 以及sidyll 在stackoverflow 上解决关于alt 的绑定的方法[^2] 的灵感，作出解决方案：

{% codeblock .vimrc %}
imap ¬ <Plug>Tex_LeftRight
imap ∫ <Plug>Tex_MathBF
imap ç <Plug>Tex_MathCal
imap ˆ <Plug>Tex_InsertItemOnThisLine
{% endcodeblock %}

问题2 稍微棘手一点，也绕了一个弯。首先参考了Mass 在tex.stackexchange上的关于设置xelatex的方法[^3] 作出如下修改：
{% codeblock .vimrc %}
function ComplieWithXeTeX()
    let oldRule = g:Tex_CompileRule_pdf
    let g:Tex_CompileRule_pdf = 'xelatex --synctex=-1 -src-specials -interaction=nonstopmode $*'
    call Tex_RunLaTeX()
    let g:Tex_CompileRule_pdf = oldRule
endfunction
map <Leader>lx :<C-U>call ComplieWithXeTeX()<CR>
{% endcodeblock %}

通过临时改变```Tex_CompileRule_pdf```这个内置参数，来调用所需引擎 。现在，问题似乎解决了，通过lx来使用xelatex，ll启动pdflatex，但是，有编号的时候，比如引用公式，则需要编译多次，不太方便，曾样才能像使用```latexmk``` 这个命令这么方便呢？在Brian 的博客[^4]的启发下，得到如下解决办法：
{% codeblock .vimrc %}
function CompileWithLatexmk()
    let targ = expand("%:t")
    execute '!latexmk '. targ
    call ErrorCheck()
endfunction
function ErrorCheck()
    let log = expand("%:r") . "." . "log"
    try
         silent execute 'vimgrep /^!/ ' . log
        copen
        catch /^Vim\%((\a\+)\)\=:E480/
    endtry
endfunction
map <Leader>lg :<C-U>call CompileWithLatexmk()<CR>
{% endcodeblock %}
加入了错误日志回复，虽然也能用了，但是和原生的插件错误日志显示完全不同，关闭会关闭整个文档。正当一筹莫展之际，灵感一现的我突然想到，如果把第一个方法结合````latexmk````，不就完美了？所以最终解决方案如下：
{% codeblock .vimrc %}
function ComplieWithXeTeX()
    let oldRule = g:Tex_CompileRule_pdf
    let g:Tex_CompileRule_pdf = 'latexmk --synctex=-1 -src-specials -interaction=nonstopmode $*'
    call Tex_RunLaTeX()
    let g:Tex_CompileRule_pdf = oldRule
endfunction
map <Leader>lx :<C-U>call ComplieWithXeTeX()<CR>

function CleanTempFiles()
    execute '!latexmk -c'
endfunction
map <Leader>lc :<C-U>call CleanTempFiles()<CR>
{% endcodeblock %}
其实就是把```xelatex```换成```latexmk```，最后还增加定义了自动清理临时文件的命令。

最后，祭出我的```.latexmkrc```[^5]，希望能帮到偶然读到的人。


[^1]: [Vim-latex User Manual 10.1 Latex-Suite Maps](http://vim-latex.sourceforge.net/documentation/latex-suite/latex-suite-maps.html)

[^2]: [Can I map Alt key in Vim?](http://stackoverflow.com/questions/5379837/is-it-possible-to-mapping-alt-hjkl-in-insert-mode)

[^3]: [How to make vim short key for xelatex and pdflatex both?](http://tex.stackexchange.com/questions/150770/how-to-make-vim-short-key-for-xelatex-and-pdflatex-both)

[^4]: [Compile with Latexmk in Vim](http://www.hesperusisphosphorus.com/blog/2014/01/25/compile-with-latexmk/)

[^5]: [My Github Gist](https://gist.github.com/skychan/153b764daf585b308356)

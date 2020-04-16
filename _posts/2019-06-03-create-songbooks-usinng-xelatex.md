---
layout: post
title: Create Song Books using XeLaTeX
description: 
summary: 
comments: true
tags: [latex, opensource]
---

Recently, I decided to digitise my songbook (lyrics with chords annotated) using the TeX typesetting system. TeX provides amazing packages for typesetting songbooks. Since my collection had mostly Malayalam songs, XeLaTeX was my choice of TeX engine which has better support for non-Roman scripts - like Malayalam, Tamil etc.

There are several TeX packages available for typesetting song lyrics and chord books. I am using the songbook package which is fairly advanced in its features. The following gist is a sample source code for generating lyrics of a famous Malayalam Melody - "തേരിറങ്ങും മുകിലെ", annotated with the chords.


```tex
\documentclass[12pt]{article}
\usepackage[chordbk]{songbook}
\usepackage{fontspec}
\usepackage{polyglossia}
\setdefaultlanguage{malayalam}
\setmainfont[Script=Malayalam, HyphenChar="00AD]{Rachana}
\newfontfamily\malayalamfont[Script=Malayalam, HyphenChar="00AD]{Rachana}
\newfontfamily\malayalamfonttt[Script=Malayalam, HyphenChar="00AD]{Rachana}
\newfontfamily\malayalamfontsf[Script=Malayalam, HyphenChar="00AD]{Rachana}

\begin{document}
	\setcounter{SBSongCnt}{1}
	\STitle{തേരിറങ്ങും മുകിലെ}{Am}
	\begin{SBOpGroup}
	  \Ch{Am}{തേരിറങ്ങും} മുകി\Ch{Em}{ലെ}
	
	  \Ch{G}{മഴ} തൂവലൊന്നു തരു\Ch{Am}{മോ}
	
	  \Ch{Am}{നോവറിഞ്ഞ} മിഴിയി\Ch{Em}{ൽ}
	
	  \Ch{G}{ഒരു} സ്നേഹനിദ്രയെഴു\Ch{Am}{താമോ}
	\end{SBOpGroup}
\end{document}
```

Running the command `xelatex songbook.tex` produces a PDF with the following output:

![Alt Text](/assets/images/xelatex-screenshot.png)

As specified in the script, it uses [Rachana](https://smc.org.in/fonts/#rachana) font released by [Swathanthra Malayalam Computing](https://smc.org.in).
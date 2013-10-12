---
layout: post
title: Installation of LaTeX packages from CTAN
tags: CTAN LaTeX TeX
---

Steps to install LaTeX packages from CTAN. This is just for future reference. This procedure may not be the best but it works.

### To install font package

{% highlight bash %}
$ unzip ec.zip
$ cd ec/src
$ tex ecstdedt				# generate driver
$ tex tcstdedt				# files
$ mf "\mode=localfont; input ecrm1000"	# to generate tfm and gf files
$ gftopk ecrm1000.%%%gf			# to pack generic font files
$ cp *.tfm /usr/share/texmf-texlive/fonts/tfm/somefolder
$ cp *.pk /usr/share/texmf-texlive/fonts/pk/ec/<resolution>/somefolder
$ cp *.mf /usr/share/texmf-texlive/fonts/source/somefolder
$ texhash /usr/share/texmf-texlive
{%endhighlight%}

### To install normal package 
Please refer this [source](http://tex.stackexchange.com/questions/30307/how-to-install-latex-zip-package-from-ctan-using-texhash-on-a-nix-system) for more information.
{% highlight bash %}
$ unzip colortbl.zip 
$ cd colortbl/
$ tex colortbl.ins
$ ctanify *.ins *.dtx
$ tar -xzf colortbl.tar.gz
$ unzip -d ~/texmf colortbl.tds.zip 
$ test -e ~/texmf/ls-R && texhash ~/texmf
{%endhighlight%}

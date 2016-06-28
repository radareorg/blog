+++
date = "2014-11-26T00:29:31+01:00"
draft = false
title = "Radare2 is documented"
slug = "radare2-is-documented"
aliases = [
	"radare2-is-documented"
]
+++
Some *miscreants* are saying that radare2 is not documented, this is wrong.

# The Book

The "radare book" was released together with radare 1.0, several years ago, so some of the examples/features may not be compatible with radare2.

You can read it [online]( http://radare.org/doc/html/contents.html ) or download the [PDF]( http://radare.org/get/radare.pdf ).

Recently, our tester in chief, maijin, started a project to update the radare book to create the [radare2 book](http://radare.gitbooks.io/radare2book/content/); feel free to contribute.

A book focused on practical case by monosource is also available : [radare2-explorations](https://monosource.gitbooks.io/radare2-explorations/content/)

# The API

The radare2 api (aka [libr]( https://github.com/radare/radare2/tree/master/libr )) is described in [vapi]( https://github.com/radare/radare2-bindings/tree/master/vapi) files. Those files are translated by [valaswig]( https://github.com/radare/valabind ) into swig interface files to generate the bindings for python, ruby, perl and others.

Those [Vala]( http://live.gnome.org/Vala ) vapi files are at the same time parsed by [Valadoc]( http://live.gnome.org/Valadoc ) to generate the [online]( http://radare.org/vdoc ) documentation.

# Articles

Some e-zines and bloggers have published articles about how to use radare.

- [phrack]( http://phrack.org/issues/66/14.html#article ), issue #66, article 14, **manual binary mangling with radare**,  by pancake
- [arteam#4]( http://issuu.com/smithcharly/docs/arteam_ezine_number4/58), **handy primer on linux reversing**, by Gunther
- [canthack.org]( http://canthack.org/2011/07/adventures-with-radare-1-a-simple-shellcode-analysis/ ), **Adventures with Radare2 #1: A Simple Shellcode Analysis**, by a *concatenation of geeks from Canterbury, UK.*
- [dustri.org]( http://dustri.org/b/defeating-ioli-with-radare2.html), **Defeating IOLI with radare2**, by jvoisin
- [crowell]( https://crowell.github.io/blog/categories/radare2/ )'s blog
- [trollprod.org]( http://thanat0s.trollprod.org/tag/radare2/ ) wrote some blogposts in French.
- [dukebarman]( http://dukebarman.pro/tag/radare2/ )'s blog - articles about Radare2 in Russian language. 
- [This]( http://radare.today ) blog ;)

# Talks
People gave [talks]( http://radare.org/y/?p=talks ) about radare2 at several well-know conferences, like [hack.lu]( http://2014.hack.lu/index.php/List#Radare2.2C_a_Concrete_Alternative_to_IDA_-_workshop ), [pses]( http://www.passageenseine.org/Passage/PSES-2014 ), [oggcamp]( http://oggcamp.org ), [rootedlabs]( https://www.rootedcon.com ), lancon, … 
We also did a lot of workshops!

# Wiki
There are some worthful information and gems on our [wiki]( https://github.com/radare/radare2/wiki ). Feel free to complete it with your favourites tips and tricks.

# Cheatsheet
[@pwntester]( https://twitter.com/pwntester ) did a really great [cheatsheet]( https://github.com/pwntester/cheatsheets/blob/master/radare2.md ) to put on your wall, along with the [refcard]( https://github.com/Maijin/radare2book/blob/master/refcard/radare2_rc.pdf?raw=true ).

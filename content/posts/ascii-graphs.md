+++
date = "2014-04-30T12:47:13+02:00"
draft = false
title = "ASCII graphs!"
slug = "ascii-graphs"
aliases = [
	"ascii-graphs"
]
+++
![graph](/blog/images/Screenshot-2014-04-29-01-51-31.png)

We may not have a GUI like IDA, but we still have some graphs. This is a small (200 lines of code) proof of concept, but there is more to come

- colors
- utf-8
- layouts
- resizing
- animations
- ...

You can try this new feature with `VV` if you are using [radare2 from git]( https://github.com/radare/radare2/ ).

![better graph](/blog/images/BmWvHyyIcAAoIfo-png-large.png)

And by the way, this is documented, and has some  tests to avoid regressions. Feel free to take a look at the [/libr/cons]( https://github.com/radare/radare2/tree/master/libr/cons ) folder if you want to contribute.
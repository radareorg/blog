+++
date = "2015-08-26T16:36:33+02:00"
draft = false
title = "chsh -s /usr/bin/r2"
slug = "r2-as-a-unix-shell"
aliases = [
	"r2-as-a-unix-shell"
]
+++
Radare2's prompt is quite powerful and handy to use, but sometimes you need to interact with the filesystem or spawn system programs.. and spawning new shells or quitting r2 is not an option.

For those cases, the simplest solution would be to just type `!` and then type the shell command you like. This prefix command will just escape to the shell and run the text you type as in the system shell.

Radare2 also implements pipes, like `pif | wc -l` to count the number of instructions in a function (careful, commands after the pipe are shell commands, not radare2 ones!). If you want to count the number of lines of a command you can also do it with the internal grep operator in r2 `~`, using the '?' modifier like this: `pif~?`.

If you want to use the result of a command inside another one, just surround it with ``` ` ```, like ``` `pi `?v 2 + 2` ```.

To run several commands in a row, you can separate them with a `;`. Now you're wondering what to do when you actually need to use a `;` in a command, like to do some ROP-gadget search. Simply double-quote **the whole command**: `"/R/ pop e[abc]x;ret"`

Sometimes, you're running radare2 in a <s>shitty</s> highy-restricted/complex environment, and you don't have a shell (for example, when you boot to radare2). This is why we have re-implemented some useful shell commands directly into radare2, like `ls`, `pwd`, `cd`, `rm`, `mv`, ... and also an internal *grep*!

You can grep on words `pdf~exit` to grep on `exit`, you can count number of line with it `pif~?`, to indent JSON like `pij~{}`, use the internal pager on the whole thing with `pij~{}..` (yes, the internal pager is `..`); but one of its most powerful feature is the ability to select by columns, with the bracket operator: `iz~[1]` to select only the first column, `iz~1337[1]` to select the first column of the matching results.

To manage your opened file, feel free to check the relevant documentation in `oo?`. The most useful commands being `oo+` to reopen the file in read-write mode and `ood` to reopen it in debug mode.

To finish this blogpost, allow us to mention the fact that radare2 has its own webserver. While this might seems super-overkill, it's actually useful to debug embedded systems, for example, a watch of an iPhone. Simply launch the web server with `=h port` and connect to it with your web-browser; you can now debug your device within the web-interface, remotely! You can also use this one liner: `r2 -qc=h /bin/ls`.

In some cases, you'll probably be interested in exposing the filesystem of the device to the outer world by using the internal HTTP server. This is done with the '=h' command, but you'll need to tune some variables to share the rootfs instead of the r2 webui:

```
> e http.root=/
> e http.bind=0.0.0.0
> e http.port=9999
> =h
```

If you don't like the webui, you can always use the `rap` protocol with `=: port`, and connect to it by issuing `o rap://9999`. Both remote interfaces can be used from the commandline of r2 by using the `r2web://`, `rap://`, `r2pipe://` and `r2 -C` connection methods.







+++
date = "2016-08-11T19:02:46+01:00"
draft = false
title = "Disassembly functionnalities inside Web UI /m"
slug = "webui-m-disasm"
aliases = [
	"webui-m-disasm"
]
+++
by [GautierGC](https://twitter.com/GautierGC)

In this blog post, we will discover the functionnalities of the disasm panel from the material Web UI and how are they implemented. This take place in the Material Web UI in place of the current implementation. If you want to read more details about the implementation and how it works, you can read [this technical article](https://blog.gautiercolajanni.fr/gsoc/r2/2016/08/11/disassembly-rewriting-and-refactoring.html) on my blog.

So, this *new* module about disassembly has the following functionnalities:

* infinite scrolling
* contextual menu 
* navigation bar (seek history)

Some parts of this module were reused from the previous module I've made: hexdump. Also, this module has a different logic about the representation of the data. Here we add some light processing to the HTML-ready content that r2 provide us. The hexdump was made around a chain to process and build the whole representation with it. They were advantages to it, here, the constraints aren't the same. I've made some refactoring on this code to authorize the re-use of some parts.

Before this implementation, scrolling was only possible inside the current displayed block, navigation was possible with two buttons. Now, this module handle an infinite scrolling to display all the content if it was all here. Everything isn't loaded with the page, there is some work done with flags/functions knowledge grabbed from analysis made with r2 and a layer to handle the communication with r2 and the dedicated web worker to let the main thread ready.

As user, you can improve the knowledge of the currently displayed data. This UI require to know about flags and functions to show useful reflinks. So, don't hesitate to press *process analysis* in the panel top bar and chose whatever suit you:

![Analysis dialog](/blog/images/webui_analysis.png)

Also, if the line allows you to process some operations, the UI will suggest you with the contextual menu to apply some commands on it (still in progress). You can currently try the following:

![Contextual menu](/blog/images/webui_contextmenu.png)

In the same logic as the browser, there is also a feature to follow your jumps. The navigation bar, or seek history allows you to know where you were and going back and forth between the addresses.

![Nav bar](/blog/images/webui_navbar.png)

These features should help to make the UI better. Don't hesistate to give us feedback about it!
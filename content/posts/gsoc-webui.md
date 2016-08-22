+++
date = "2016-08-20T17:40:41+01:00"
draft = false
title = "GSoC WebUI overview"
slug = "gsoc-webui-overview"
aliases = [
	"gsoc-webui"
]
+++
by https://twitter.com/GautierGC

April 22th, my GSoC proposal for the radare web UI [was accepted] https://summerofcode.withgoogle.com/organizations/4965722304282624/#5484657504157696. Until today, I've made the UI progressed. The purpose of this article isn't to close this chapter but to relate what I've done and share my experience about this Google Summer of Code.

I would like to make a short overview of what I've done, a big picture of what I improved during the last months. More details are available on my [GSoC related posts](https://blog.gautiercolajanni.fr/gsoc/). The official beginning (coding part), has started May 23th and will end August 23th.

Proposal period
---------------

When I started to take a look to this project. There were a lot of code duplication due to the lack of dependency management. As you may know in his [current repository](https://github.com/radare/radare2-webui), radare count 4 UIs. I've managed to centralize dependencies into one place and add bower and gulp to manage external dependencies.

Community bonding period
------------------------

During this part, I've continued to improve the management of the repository through several ways. This period, about 1 month, has allowed me to normalize some items:

* adding last dependencies (fonts) to gulp
* adding code climate badge to visualize code quality
* cleaning the repository from unused files
* updating the r2pm update process (the distribution has changed)
* adding: README, CONTRIBUTING and LICENCES files

With better dependency management, we can now easily what we are using and update them. Adding clode climate has allowed us to visualize where were the bad things and determine a regular check style for the whole repository. The new r2pm package allows us more flexibility. Finally, the files about README, CONTRIBUTING and LICENCES allows users and contributors to know a little better how does radare Web UI's are working.

Main part
---------

From my point of view, there were no transition between the previous part the main coding part. I was into the project, ready to dive into the code base that I've discovered during the previous weeks.

End of may, I was finalizing how the UIs would be distributed and how I and every contributor would be able to make them work. We were moving from a simple makefile to a more complete dependency mangement tool who now support: external libraries, packaging (releases), development time through a watcher and a livereload plugin. Then, I've started to make *true* code. In the following description, I will not talk about *small fixes* but more about new functionnalities or major rework.

So, **first task**: the autocompletion module into the main search field (top right corner). I've made a simple form with a dropdown to allow the user to visualize the different options. The purpose isn't to complete the request but to select the element where you want to go. It's generic enough to be bound to another field and be fed with another r2 command.

**Second task**, more about network error that we can encounter between r2 and the UI. We are able to indicate to the user that something has gone wrong and, at the same time, allowing the user to retry. For this part, we used a *callback* plugged into `lib/r2.js` to open a dialog. I've made the choice to use the native dialog from HTML5 standard. Not well supported by every browser, it's currently backed up by a polyfill.

Then, I've started the **third task** about the presentation of big tables. Or more generic, how to present a big amount of sortable data. Here, I've made the choice to use this tool: [DataTable](http://datatables.net/). Fit perfectly the needs: sorting and filtering data. One small problem: integration with [MDL](https://getmdl.io/) is not perfect. Globally, the integration into the current Web UI was pretty easy with very few rework, just an adapter to bind data to this tool. All tables are currently handled by this tool.

Next step involved the improvement of the status bar. It was my **fourth task**. It's actually pending, *work in progress*, there were several attempts to make it usable, fully integrated with the rest of the UI. It will need some rework to include all the tools we want to see into this bar and make it more context-aware.

I've got a blank period during the **fifth task**, I had some exams and homework to deliver to my school. I, then, pushed a very specific improvement to this current UI: the ability to split the screen without *iframe*. It allows us to keep the same context and open the possibility to a composable UI. We consider the previously called *panels* as *widgets*. There is still work to do to make it more stable and to allow future usage but it currently permits to split the screen vertically. There were several challenges due to refreshing functions defined more global than required and how [MDL](https://getmdl.io/) was using the *flexbox* capacity.

The **sixth task** was, in time invested into, the biggest. I've made a new implementation the previous hexeditor without the formating help of r2. It took me nearly one month to develop this but it includes a lot of new functionnalities. A lot of them were reused into the disassembly panel (seventh task). The main sub-tasks are the following:

* infinite scrolling
* (partial) selection
* dedicated contextual menu
* flag positionnement and colorization
* inline edition (ascii or pairs)
* export selection with r2 processing

I've also encountered some performances issues with this panel. Fetching data is something that happen synchronously and the first processing take also some time. Since browsers aren't able to isolate background processing into an other thread, I've made use of Web Workers to be able to make some processing in the background and let the main thread available for user's actions. You can read the [dedicated article](https://blog.gautiercolajanni.fr/gsoc/r2/2016/07/15/webwokers-hexdump.html) for more details.

For the **seventh task**, I've made a rewriting of the disassembly panel by using the previous tools. I've started to clean them and refactored some of them to be more suitable for this task. The rewriting is also important but I'm using more content from r2 and I reused some components I've developped for the hexdump editor. Currently, the disassembly panel handle:

* infinite scrolling based on flag/functions knowledge
* dedicated contextual menu
* apply commands and retrieve result through dialog

And then, I've reached the end of this GSoC. My **last tasks** were some issues referenced into the repository and bug fixes spotted into the panels. It includes some small improvement like highlighting the currently seeked line into disassembly or merging the header panel into the overview panel using tabs.

Original plan deviation
-----------------------

From my point of view, I didn't miss the main objective of my proposal: make the UI better. But, I can't deny I'm far away the initial proposal I've made.

I've defined some major goals: search function, hexdump, disasm, keyboard shortcut, dual panel, graph module, interactivity through T command and scripting area. From this point of view, I haven't filled out all those tasks. I think I've been a bit too optimistic when I've placed them into my schedule. For example, I've considered that the hexdump should have taken me one week, then I spent one month.

So, if somebody would have to complete this proposal, he/she should take care of:

* color theme customization
* configuration of global keybinding
* improve vertical dual-panel with horizontal split (or more)
* incorporate and make accessible a better graph module
* add interactivity through T command
* improve the scripting area
* improve debugger interface (breakpoints, backtracking, etc.)

It's not exhaustive, I think the most important here is to make something usable and used by the community more than a copy of the CLI.

Future
------

This experience was very interesting and I don't want to stop here. There is plenty of work to do to improve this UI. It will be tough to raise this UI to the command line level and I don't think it's the right way to build a great UI. We will need feedback from the community to focus on the most used features and the one that miss the most.

In september, I'm beginning an internship but I don't want to let this project alone. I will have less time to work on it but I'm very interested to make it evolve.

I will have the time to take a step back, visualize in which direction the UI is going, what users wants. There is also some technical challenges, how to handle the *big ball of mud*, improve modularity, reusability and continue to consider performances and usability issues.

Last words
----------

The Google Summer of Code is a great experience. The radare team has really been present to help me implement what I had committed myself and had helped me to obtain the right informations to interface the UI with r2 despite my lack of knowledge about this tool.

I would like to thanks my mentors [Álvaro Felipe](https://twitter.com/alvaro_fe) and [pancake](https://twitter.com/trufae) for their help during my GSoC, but also [Maijin](https://twitter.com/Maijin212) and [Anton Kochkov](https://twitter.com/akochkov) for their time and their knowledge about r2. It was a big opportunity for me and I hope that my work will help the r2 community. In every case, I will keep following this project and continue to contribute.
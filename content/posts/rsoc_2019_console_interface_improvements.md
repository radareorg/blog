+++
title = "RSoC 2019 Final: Console Interface Improvements"
aliases = ["RSoC 2019 Final - Console Interface Improvements"]
date = 2019-10-01T14:10:05+05:30
slug = "rsoc-2019-console-interface-improvement"
draft = false
+++

# RSoC 2019 Final: Console Interface Improvements

### Introduction:
Hello all, I’m [deepakchethan](https://github.com/deepakchethan) from India. I got to work on the [console interface improvements](https://www.radare.org/rsoc/2019/ideas.html#title_0) for radare2 as a part of 2019’s edition of [Radare Summer of Code](https://www.radare.org/rsoc/2019/ideas.htm). My main task was to improve the terminal interface of radare2. As a part of which I was tasked with completing 6 main tasks. I was unable to complete the table API myself, gladly pancake helped me with implementing the Table engine, while I worked on the integration and various improvements of it. I learned a lot in the 3 months and am very grateful to my mentors Pancake and XVilka for being flexible and supportive in all aspects of the program.

### Task 1: Generalize the code of popup widget:
Radare2 already had vim-like autocompletion in the visual offset prompt. My job was to generalize it and integrate it with the RCons. This feature can be enabled by setting `scr.prompt.popup`. It is still a little buggy but the result is as follows:

![popup](/blog/images/rsoc_popup.png)

### Task 2: Improving autocompletion and dietline modes (vi/emacs-like hotkeys)
Autocompletion was not supported for k and other mount based commands. I was tasked to add them. I also integrated dietline into *mount shell* (ms) and *HUD* mode.

![hud](/blog/images/rsoc_hud.png)

I added the missing emacs hotkeys and added vi modes, which can be enabled by setting `scr.prompt.vi` to `true`. As a bonus if you set `scr.prompt.mode` to `true`, the color of the prompt changes based on the vi mode (control/insert). See the exact list of the hotkeys in [Dietline](https://radare.gitbooks.io/radare2book/content/basic_commands/dietline.html) Radare2 book chapter.

![vi](/blog/images/rsoc_vi.png)

I also added support for some bash commands like `uniq`, `sort`, `join`, etc.,

### Task 3: Support color-scheme in radiff2 graphs
My next task was to add support for ASCII/Unicode graphs for radiff2. But I noticed that the `agd`-based diff commands were not implemented in radare2. So I began with that, once it was complete, exporting them to radiff2 was relatively simple. Now, radiff2 supports all these graph outputs:

 - ASCII or Unicode (UTF-8) art
 - r2 commands
 - Graphviz dot
 - Graph Modelling Language (GML)
 - JSON
 - JSON with disasm
 - SDB key-value
 - Tiny ASCII/Unicode art
 - Interactive ASCII/Unicode art

The generated ASCII diff graph between true and false:

![radiff](/blog/images/rsoc_radiff2_graph.png)

### Task 4: Further improvements for UTF-8, BiDi:
All the reflines in disasm were of the same kind, which was a bit confusing. So I added different colors for up and down going reflines. The new look:

![refline](/blog/images/rsoc_refline.png)

I also added UTF8 support for line graph commands like `p=` and `p==`.

![p==](/blog/images/rsoc_p=.png)

### Task 5: Implement table commands and API (like it is done for graphs)
I was not able to implement the table API, but pancake did it. So I was tasked to add UTF-8 support and integrate the table API in the various commands that will benefit from the table representation. The `p-h` command integrated with the table API looks as follows:

![Table](/blog/images/rsoc_table.png)

### Task 6: Various bugfixes and improvements for visual, visual panels and graph modes
I fixed quite a bunch of bugs and all of these can be found [here](https://github.com/radareorg/radare2/pulls?utf8=%E2%9C%93&q=is%3Apr+is%3Aclosed+author%3Adeepakchethan+).  
	





+++
date = "2019-02-02T00:00:00+00:00"
aliases = [ "Radare2 Community Survey Results" ]
draft = false
slug = "radare2-survey"
title = "Radare2 Community Survey Results"
+++

Overview
--------

As part of our efforts to make radare2 as relevant as possible for our community, we decided to involve our users in the decision-making process. A few weeks ago we published a short survey, collecting different questions from wide range of types. We asked our users to choose their preferences for different commands, what would they prefer for developers to focus on, and even what makes them prefer other tools over radare2. Most of the questions were with closed answers, but some of them -- in which we wanted to let the community express themselves -- were open.

More than 80 users answered within the first hour and overall we collected 153 answers to the survey. We weren't expecting such big numbers and were so happy to get all this feedback from you. Your kind words were very touching that we were so excited to read all this positive feedback and your love for the project. Such appreciation is our drive to make radare2 better.

In the following blog post, we will try to summarize the main critics you've heard as well as misconceptions and features we would like to highlight and remind.

Overall, we want to say a BIG THANKS to everyone who filled the survey. We care about your comments! We will publish more surveys in the future, so stay tuned and follow us on social media to get to know when is the next survey.


![r2survey_dev_focus](/blog/images/r2survey-dev-focus.png)


Cutter - Our Graphical User Interface
----------------------

One of the most interesting questions we asked you was "What would you like that r2 devs will focus more on?". The most chosen answer was Cutter -- our GUI. There is no doubt about it, a large number of users are interested in better GUI. [Cutter](https://github.com/radareorg/cutter) is radare2's official Graphical User Interface and has its own dedicated development team. Cutter consistently improving in terms of speed, stability, and new features. Want to help? The Cutter team would be grateful to hear your feedback and will happily welcome new developers to the team.

Make sure to follow the project on [Github](https://github.com/radareorg/cutter) and on [Twitter](https://twitter.com/r2gui).

**Your comments:**

Many of your comments asked us to focus more on Cutter:

> focus more on "GUI that exports more capabilities of r2"

> invest "more effort on the GUI"

Due to the increasing demand for a decent GUI for radare2, Cutter is indeed one of our top priorities. A lot of efforts is invested in developing Cutter, its smooth integration with radare2 and export the huge amount of capabilities from radare2 into a clean, fast and stable GUI.

You also said that:

> Cutter seems to use a lot of memory on Windows compared to the radare2 command line. 

Naturally, the radare2 command line is much lighter than a modern GUI. Creating a multi-threaded, well-designed GUI isn't an easy task and it requires many widgets and background radare2 API commands to be executed simultaneously. That said, there is a lot of work left to do in Cutter in order to make it faster and less memory-hungry. In the recent weeks, Cutter had been through many improvements that made the GUI much faster and sophisticated. The few next weeks would bring even more improvements in performance and efficiency.

Signatures
----------

It was always on our roadmap to bring a proper auto detection of library functions to radare2. In this survey, we received a lot of comments by you, asking us to improve the support of out-of-the-box library functions detections. While our framework already supports signature matching and pattern recognizing from different types, we don't ship signature with radare2 -- and this is about to change.

**FLIRT**

You asked for:
> IDA like function identification (FLIRT)

> Library detection missing like flirt

> ...things like FLIRT that works out of the box...

Different answers asked us to support FLIRT Signatures, the truth is - that radare2 is already supporting it and you can convert IDA's FLIRT signatures into radare2.


**Signature Shipping**
As mentioned above, it was clear that our community want us to ship by-default signatures for popular libraries:

> Missing zignature proper support and packages with ready to use zignatures.
  

> Please ship library signatures with r2. It's the best when you load a random binary and so many functions are already detected by r2

 
> Lack of library detections support because zignatures isn't as easy to use. (CC how IDA is doing this) 

> Please focus on... more detection to library functions 

This request was so popular that we decided to highlight it and give it a higher priority on our roadmap.

To sum up, it is somewhat unknown to our users that radare2 supports both IDA Pro FLIRT and its own signatures format. The only problem is missing the standard pack of ready-to-use signatures for main libraries. We hope to improve this feature and make radare2 and Cutter ship popular library signatures by default.

We suggest everyone to check out the [Zignatures chapter](https://radare.gitbooks.io/radare2book/signatures/zignatures.html)
from r2book for more details.


Decompiler
----------

![r2survey_decompiler](/blog/images/r2survey-decompiler.png)

Decompilation is not an easy task, and as in many aspects of radare2 we have integrated several decompilers with r2, but this is not a core feature of r2. Same goes for a GUI, so those tasks have been delegated to radeco/r2dec and Cutter which are developed under the radareorg umbrella. We can improve the integration with those tools, but the quality of the decompilation doesn't depend on r2, so feel free to contribute or fill issues in those projects which run on top of r2.

We also support popular decompilers such as Retdec and Snowman as external plugins that are available from r2pm, our package manager. We have plans to improve the support for retdec and we will appreciate any help with this.


Documentation
-------------

We were happy to read that you want to know more about radare2 rich functionality and the inner structure of the radare2 code base. You also showed high interest in exploring more about radare2's capabilities by reading documentation and blog posts:

> We need more showcasing with practical examples like what @Megabeets does; I've been an r2 user for about a year and I am dead sure I only know ~10% of r2

> I'd like to see some blogs on how r2 devs approach some issues so people can get motivated to contribute

> I like the idea of having explicitly described info, I get why some may find that undesirable, but I think to ease the learning curve of r2 it doesn't hurt to be descriptive.

> I want you to enrich radare2book more.

Since radare2 is a community project, we invite our community members to add more descriptive information, and to write blog posts and writeup explaining how to use radare2. You are all welcome to get help in our Social Media profiles and on our Telegram & IRC groups. If you want to publish a writeup on our blog, please feel free to tell us.


Scripting
---------

There were also some comments about scripting, which we comment below:

* Can't use r2pipe to make an intern plugin

r2pipe is a basic communication protocol to run commands and get the output, nothing else. With that in mind, people should know about `r_lang`. This library provides a way for scripting languages to communicate natively with r2.

Initially, this library just provided a way to get the pointer to the current `RCore` instance, and from this, just using the native bindings of the language it is possible to do anything. The current problem is Building `r2lang-python` and `r2bindings-python` is usually too much work for most users and as long as the bindings are unmaintained because almost nobody uses them, pancake implemented a thin plugin layer inside the language plugin for r2.

This is called r2lang, and it's available right from the `r2lang-python` (as well as for any other languages) by just `import r2lang`. This module contains methods to construct native plugins from Python. and also allows you to use r2pipe using the native API, which is the fastest method for interacting with r2.

So well, you can do core, asm, anal, bin, io and core plugins with that API. See some documentation from this r2m2 talk and the r2 book in here:

* https://guedou.github.io/r2m2_talks/2016_r2con/slides.pdf
* https://radare.gitbooks.io/radare2book/plugins/python.html

Another complaint was about having the ability to interact with the target process as well as having an async API to receive events.

* async trace API for scripting

That's something we thought already, and there is a four years old issue waiting for feedback on this in order to get the right design before implementing it for all the languages.

Projects
--------

The ability to save your work in Projects is one of the highest demanded features by the radare2 community. While the feature was implemented in radare2 a long time ago, it suffers from major bugs that make it hardly usable.

We aware how important Projects are for the community and fixing the issues is on our roadmap. We use Github-Projects to log our process and things-to-do, you can check it [here](https://github.com/radare/radare2/projects/9). Meantime, as always, we appreciate any help we can get - please report your problems in our issue tracker and if you want to help us with some coding - please check the Projects and RBin projects in radare2’s Github page. They describe most of the needed changes required to get Projects to work in a stable way, and at the same time enable Real-Time Syncing and Undo-Anything feature.

pancake draw the plan to fix projects 2 years ago, but this requires many refactorings in order to get them to work properly, we can do a quick fix for most use cases, but the level of complexity of tasks you can achieve with r2 can't be solved that way. So, if you care about projects join the chat and help us squash some of those requirements because we would take any help offered.

Better Windows Support
-----------------------

Radare2 supports native debugging on Windows machines, in addition to extensive support of static analysis of the PE file format. We are always working on making our framework better so if you have feature-requests and issues, please report them in our Github page -- This is highly important for us.


Useless Features
----------------

One of the replies we got said:
> Stop adding new features and focus on maintaining and improving existing ones.

This is a complaint we have already heard before, and it's probably a misunderstanding or a lack of knowledge of what we (the radare2 developers) do every day for the project. Most of our commits are related to improving the stability, speed up, make more consistent and test better the actual features of r2.

The reasons why users complain about this are probably some of the following:

* Coe features which aren't UI are not something that easy to show on tweets and with shuny graphics. This is why, some of our most important improvements aren't getting enough attention.
* Different users have different priorities. We all use radare2 for different things and a feature which is very important for one, can be entirely useless for another.
* Issues are not reported to our bug tracker on Github. We can't fix an issue we don't know about.
* Many new features are implemented in order to set the base for implementing other important capabilities.

Final Words
-----------

It was incredibly nice to read all your feedback and in fact, it was that nice that the next week IDA and Binary Ninja did a survey too :P 

We know radare2 have still a long path for excellence, and our development priorities depend on our understanding of the tool and personal needs. Not everybody solves the same problems or use the same tools (not even in the same way). And radare2 is one of the most orthogonal, complex and extensible piece of software you may find out there nowadays.

We do constant refactorings on every piece of the program, this gives us the ability to improve in a more consistent way without having to maintain badly designed APIs that just confuse developers and make the whole thing a mess.

There are still so many things to discuss and get your feedback and we plan to make more refined surveys to embrace the community in an anonymous way and show the reasons behind each decision.


Thank you very much!

+++
date = "2017-03-12T17:30:00+01:00"
draft = false
title = "Project files"
slug = "project-files"
aliases = [
    "Project-files"
]
+++


# Project files

**Disclaimer**: Projects files are highly subject to change but here is the current state on March 14th 2017. The feature is still under high work in progress please see the section Future of Project Files and How to Help.

![CaseXXXX](/blog/images/project_intro.png)

# Purposes

The Project files are used to store information related to your radare2 session. The idea is to use this Project file to save your analysis for later work, share it with an audience (colleagues, friend, RE articles...), and scripting.

# How it look like in radare2

## How to use

In radare2 world, you can manipulate the Project files using the `P?` set of commands and `r2 -p proj myfile` command.

To save a Project, you can use `Ps name` (Project Save) in a radare2 session, this will trigger the save of the project. By default, the config files do not contain your file, only information about the radare2 session.

To reopen your project, simply do `r2 -p name myfile` or `Po name` in the interactive session.

Projects are stored in ~/.config/radare2/projects by default. (Can be changed via `dir.projects`)

`Pn-` allows you to edit project notes.

## File Format

A project file is usually consists of an `rc` file and some `.sdb` files.

The `rc` file contains plaintext radare2 commands.

For example here are the flags:

![](/blog/images/project_flags.jpg)

The sections:
![](/blog/images/project_sections.jpg)

The Configuration:
![](/blog/images/project_eval.jpg)

But many other things are stored such as analysis, symbols etc. I strongly encourage you to look at the content of the rc files to get a better understanding of radare2 itself.

The SDB files are databases in [sdb](https://github.com/radare/sdb) format.

## The Evals

You currently can configure the project file via the following eval variables:

* `e prj.files = true`: Save the target binary inside the project directory
* `prj.name = true`: Name of current project
* `prj.zip = true`: Use ZIP format for project files

### Git Versioning

A recently added feature allowws you to 
* `e prj.git = true`: Every project is a git repository.

Currently saving the project via `Ps` will prompt you for a commit automatically.

Here is an example of this in use [malware analysis article](http://radare.today/posts/malware-static-analysis/).

See the [commits](https://github.com/Maijin/caseXXXX)

![Case](/blog/images/project_case.jpg)

### Storing commands in your project file

You can insert radare2 commands into your project file, that can be executed from the script with `Pnx`. This is useful if you have a set of commands that need to be run to bring radare2 into a specific state each time you start working on your project.

Any line prefixed with a `:` is considered a command and will be run when you call `Pnx`

# Future of Project Files and How to Help

We currently have a META - Issue grouping all the project related issues [here](https://github.com/radare/radare2/issues/6945). 

It contains a list of fixes/enhancements that you are welcome to help fixing.

In addition to better reliability, one of the functionality we are currently looking after is the possibility to sign and encrypt project via GnuPGv2.




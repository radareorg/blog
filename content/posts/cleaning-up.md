+++
date = "2014-05-07T00:36:29+02:00"
draft = false
title = "Cleaning up"
slug = "cleaning-up"
aliases = [
	"cleaning-up"
]
+++
By default `sys/install.sh` puts everything under /usr. Just to make things easier There are several reasons for this, but it may polute your system if you install multiple versions of r2 or use the one contained in the package system of your distro.

If you want to remove previous installations of r2 from a specific directory type the following commands:

```
$ ./configure --prefix=/usr/local
$ make purge
```
The `purge` will remove all r2 files from current and previous installations (older versions of it) from the /usr/local directory.

In addition, sometimes, when building from `git` there are stale .d files which are included by the `make` by causing the build system to try to link to unexisting object files.

The way to clean your local git repository of r2 just run the following command:
```
$ git clean -xdf
```
This command will remove all non-tracked files from the local copy and should let the build system to run happily again.

```
$ sys/install.sh
```

If you had local changes that break the git pull and want to get rid of them. Instead of cloning the whole r2 repo again just do the following:

```
$ git reset --hard @~10
$ git clean -xdf
$ git pull
```

Hope those hints let new users understand some more things about Git and how the build system of r2 works.
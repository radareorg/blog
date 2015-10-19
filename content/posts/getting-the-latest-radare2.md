+++
date = "2014-05-09T23:31:43+02:00"
draft = false
title = "Getting the latest radare2"
slug = "getting-the-latest-radare2"
aliases = [
	"getting-the-latest-radare2"
]
+++
Since radare2's developement is pretty quick, the recommended version is the current git, and not the stable one. At least if you want to play with it in a comfortable way.

You can always install it from your favorite packet manager if you are lazy: we are packaged in a lot of distributions.

## Simple way
```
$ git clone https://github.com/radare/radare2.git
$ cd radare2
$ ./sys/install.sh
```
And that's it, radare2 will be 

1. Updated
2. Compiled
3. Installed on your system

To update it, just run `./sys/install.sh` again.

## In a container
Since [docker]( https://www.docker.io/ ) is awesome, here is the receipt to get radare2 inside it, with every bindings.

```
# using phusion/baseimage as base image. 
FROM phusion/baseimage:0.9.9

# Set correct environment variables.
ENV HOME /root

# Regenerate SSH host keys. baseimage-docker does not contain any
RUN /etc/my_init.d/00_regen_ssh_host_keys.sh

# Use baseimage-docker's init system.
CMD ["/sbin/my_init"]

# create code directory
RUN mkdir /opt/code/
# install packages required to compile vala and radare2
RUN apt-get update
RUN apt-get install -y software-properties-common python-all-dev wget
RUN apt-get install -y swig flex bison git gcc g++ make pkg-config glib-2.0
RUN apt-get install -y python-gobject-dev

# compile vala
RUN cd /opt/code; wget http://download.gnome.org/sources/vala/0.24/vala-0.24.0.tar.xz; tar -Jxf vala-0.24.0.tar.xz
RUN cd /opt/code/vala-0.24.0; ./configure --prefix=/usr ; make && make install
# compile radare
RUN cd /opt/code; git clone https://github.com/radare/radare2.git; cd radare2; ./sys/all.sh

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```
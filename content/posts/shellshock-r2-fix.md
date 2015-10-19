+++
date = "2014-09-30T02:08:29+02:00"
draft = false
title = "Shellshock r2 fix"
slug = "shellshock-r2-fix"
aliases = [
	"shellshock-r2-fix"
]
+++
A lot have been discussed recently about this [vulnerability]( https://shellshocker.net/ ) in bash. The internet was totally shocked just like what happened with [heartbleed]( http://heartbleed.com/ ).

Several vulnerabilities have been discovered in bash, which caused the distros to release several updates on the same package and being a little chaotic to know which was the correct patch to take.

The vulnerability was not just affecting local users which have a little risk, but webservers running CGIs because http parameters are passed as environment variables to the script which was executing the functions defined in there.

Finally, it seems that the tempest is calming down and everyone is happy again with the GNU Bourne Again Shell.

But vulnerabilities in security are always not correctly patched, or even properly updated all the affected systems. This is the case of critical servers which can't be easily updated or require too much manual intervention like compiling a new version of the shell by hand, etc.

On embedded systems, this is probably the worst place, because they tend to be poorly updated, luckily few embedded systems use bash for memory constrains reasons. But the vulnerable systems are still there.

At the moment of writing Apple didnt released a security update to fix that vulnerability on OSX. So this exposes all the OSX users and servers.

UPDATE: Apple just released an [update]( http://arstechnica.com/apple/2014/09/apple-patches-shellshock-bash-bug-in-os-x-10-9-10-8-and-10-7/ ) while I'm writing that blog post.

The binary patch
------------

After reading some links I found some people doing [binary patching]( http://www.openwall.com/lists/oss-security/2014/09/29/1 ) for fixing the vulnerability (by removing the functionality).

As the comment describes, the patch consists on nullifying the first byte in every search result of '() {\x00'.

Solar Designer has implemented a Perl oneliner to fix it which is described on [Bruce's blog]( https://www.schneier.com/blog/archives/2014/09/nasty_vulnerabi.html#c6679473 ) and also twitted [here]( https://twitter.com/solardiz/status/516370924426514433 ).

	perl -pe 's/\(\) {\0/(){\0\0/g' < /bin/bash

This is a good oportunity for showing how to do this with r2 and explain what the patch does.

	r2 -qnwc '/ () {\x00;wx 00@@ *' /bin/bash

In essence the patch length is the same as in Perl. that give us a quite good compression ratio :)

Let's explain what the r2 oneliner does:

Flags:
----
* `-q` : quit after running all the -c commands
* `-n` : do not load rbin information
* `-w` : open the file in read-write mode
* `-c` : run the following command

Commands
------

* `/ () {\x00` : searches the string `() {` finalized with a null byte. 
* `wx 00 @@ *` : writes a null byte (Write heX 0x00) at the very first byte of each search hit.

Patching OSX
---------
I have tested this in my local Mac and it worked as expected:

    $ /bin/bash --version
	GNU bash, version 3.2.51(1)-release (x86_64-apple-darwin13)
	Copyright (C) 2007 Free Software Foundation, Inc.
	$ cp /bin/bash .
    $ env x='() { :;}; echo vulnerable' ./bash -c ":"
    vulnerable
    $ r2 -qnwc '/ () {\x00;wx 00@@ *' bash
    $ env x='() { :;}; echo vulnerable' ./bash -c ":"
    $

If we look closer of what that patch is doing we will notice that in OSX it will be patching that null byte in two places. This is because that bin is a fatmach0 that contains the x86-32 and x86-64 programs. We can extract them with this command:

    $ rabin2 -x bash
	bash.fat/bash.x86_64.0 created (612336)
	bash.fat/bash.x86_32.1 created (609744)

Here we'll see that each file contains only one result for searching "() {\x00".

Understanding the patch
-----------------
The patch is chopping a string in an array of them which is used to permit the function definition support on environment variables.

That binary patch is just disabling the functionality that [this patch]( https://ftp.gnu.org/pub/gnu/bash/bash-3.2-patches/bash32-054 ) is fixing .

That will make the STREQN conditional to always be true.

As long as this is an exotic functionality I guess the patch will not break any bash script. Anyway it's always good to keep a copy of the original binary before patching it to compare or be able to go back to the previous state.

Other vulnerabilities
---------------
Other vulnerabilities has been also found in bash caused by not correctly fixing the problem or similar ways to exploit the bug.

You can find them all at [here]( https://shellshocker.net/). I'll list them and mark them as FIXED if the patch fixes the problem (tested on OSX)

Should not say "vulnerable"  (FIXED)

	env x='() { :;}; echo vulnerable' bash -c ":"

Should not display the date  (FIXED)

	env X='() { (shellshocker.net)=>\' bash -c "echo date"; cat echo ; rm -f echo

Should not say "vulnerable" (FIXED)

    env -i X=' () { }; echo vulnerable' bash -c 'date'

Should not crash (NOT AFFECTED)
    
	bash -c 'true <<EOF <<EOF <<EOF <<EOF <<EOF <<EOF <<EOF <<EOF <<EOF <<EOF <<EOF <<EOF <<EOF <<EOF' || echo "CVE-2014-7186 vulnerable, redir_stack"

Shold not display the CVE number (NOT AFFECTED)

    (for x in {1..200} ; do echo "for x$x in ; do :"; done; for x in {1..200} ; do echo done ; done) | bash || echo "CVE-2014-7187 vulnerable, word_lineno"

You can find a script that collects all the patches for bash on [this]( https://github.com/tjluoma/bash-fix/blob/master/bash-fix.sh ) github. 

--pancake
+++
date = "2014-06-11T12:59:21+02:00"
draft = false
title = "Who uses r2 ?"
slug = "who-uses-r2"
aliases = [
	"who-uses-r2"
]
+++
Everyone knows [IDA]( https://www.hex-rays.com/products/ida/ ) and [Ollydbg]( http://www.ollydbg.de/ ), but not everyone has [2700€]( http://rada.re/r/cmp.html ) to spend on a software, nor wants to trust/use closed-source applications.

But who uses radare2 as a replacement ?

## Cool projects
Some reverse-engineering/security-oriented projects are using radare2, thanks to its [convenient license]( https://github.com/radare/radare2/blob/master/COPYING.LESSER ) (GPL/LGPL).

- some [coreboot]( http://www.coreboot.org/ ) developers are [using radare2]( http://wiki.bios.io/doku.php ), since it supports not only x86
but also 8051, H8, CR16, ARM, used as [embedded controllers]( http://www.coreboot.org/Embedded_controller ).
- [Droid Developers / MILEDROPEDIA]( http://droid-dev.mobi) using radare2 for the reversing baseband DSP firmware/RTOS ([TMS320C55x+]( http://droid-developers.org/wiki/Wrigley_3G ) architecture, unsupported in IDA Pro).
- radare2 is included in [Kali]( http://www.kali.org/ ), [ArchAssault]( https://archassault.org/ ) and the [Remnux]( http://zeltser.com/remnux/ ) Linux distributions.
- The [malware must die]( http://malwaremustdie.org/ ) team [is]( https://twitter.com/MalwareMustDie/status/479034806592745472 ) [using](  https://code.google.com/p/malwaremustdie/wiki/MalwareMustDie_RemnuxTips ) [radare2]( http://blog.malwaremustdie.org/2012/09/slight-changes-in-shellcode-dropper.html ).

## Wargames
Since radare2 has some useful features for the exploit hunter/developer, it was expected to be found here.

- The well known [IO wargame]( http://io.smashthestack.org/ ).
- Every [overthewire]( http://overthewire.org/wargames/ ) wargames.
- [Root-me](http://www.root-me.org/en/Challenges/App-System/) App-System challenges.

## IT security companies and researchers
Since radare2 is open and scriptable, it is slowly being adopted as a malware reversing and classification tool.

- [Alien Vault]( http://www.alienvault.com/open-threat-exchange/blog/osx-leveragea-analysis ) is using radare2 and they even did a [workshop]( http://multimedia.telos.com/blog/how-can-collaborative-threat-intelligence-benefit-you ) about it at Blackhat!
- [Bloomcraft]( http://bloomcraft.net/home/careers ) is mentioning it on their career page.
- [Craig Heffner]( http://www.devttys0.com/ ) from [Tactical Network Solution]( http://www.tacnetsol.com/ ) mentionned it during its [Blackhat talk]( http://www.devttys0.com/wp-content/uploads/2014/04/FindingAndReversingBackdoors.pdf ).
- [nitr0usmx]( https://twitter.com/nitr0usmx ) from [IOActive]( http://ioactive.com/ ) wrote a binary diffing [article]( http://chatsubo-labs.blogspot.fr/2013/10/binary-diffing-visual-en-linux-con.html ) about radiff2 and they  discuss radare2 during their [Blackhat 2012]( https://media.blackhat.com/bh-us-12/Briefings/Santamarta/BH_US_12_Santamarta_Backdoors_Slides.pdf ) talk.
- [Pau Oliva]( http://pof.eslack.org/ ) from [viaForensics]( https://viaforensics.com/ ) wrote an [article]( http://pof.eslack.org/2014/04/08/hacking-super-street-fighter-ii-turbo-part-2/ ) about how to hack MAME with radare2.
- [Maijin](https://twitter.com/maijin212) from [FireEye](https://www.fireeye.com/).
- [Futex](https://twitter.com/futex90) from [Malware.lu](http://www.malware.lu) wrote an [article](https://connect.ed-diamond.com/MISC/MISC-083/Chasse-aux-malwares-sous-GNU-Linux) about usage of radare2 for malware in French ITSec magazine "MISC".


## Talks
A lot of talks about radare2 were given in various places: [rootedcon]( http://rootedcon.es/ ), lacon, [blackhat]( https://www.blackhat.com/ ),  [phdays]( http://www.phdays.com/ ), nopcon, [owasp]( https://www.owasp.org/index.php/Main_Page ), ncn, campus party, summercamp, [fiberparty]( https://twitter.com/fiberparty ), ...

You can get slides [here]( http://radare.org/y/?p=talks ).

## CTF teams
[Capture the flag]( https://en.wikipedia.org/wiki/Capture_the_flag#Computer_security ) is a competition that is composed of a number of security-related challenges like exploitation and reverse engineering. If you're dealing with convoluted and exotic binaries, radare2 is the right tool for you!

- The [Dragon Sector]( http://blog.dragonsector.pl/2014/04/plaid-ctf-2014-tiffany-writeup.html ), one of the [best ctf teams]( https://ctftime.org/team/3329 ) in the world.
- The [LSE]( http://blog.lse.epita.fr/articles/14-defcon2k12-prequals-pwn300-writeup.html )
- The [Sexy Pandas]( http://nopsr.us/ctf2008qual/rev500.html )
- [Shellphish]( http://shellphish.net/ )
- [Hackgyver]( http://www.hackgyver.org/ )
- [Madhaxers]( https://www.facebook.com/Madhaxers )

Feel free to tell us if you are using radare2 and want to be listed on this page.

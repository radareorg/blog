+++
date = "2018-08-31T00:00:00+00:00"
aliases = [ "Musings on adding more biology formats into the radare2 reverse engineering framework" ]
draft = false
slug = "radare2-bioinformatics"
title = "Radare2 and bioinformatics: a good match?"
+++

## Intro

Ahead of this [years' radarecon][radarecon2018], pancake nudged me into discussion we both have about how [software reverse engineering][reddit_reveng] and bioinformatics compare and might complement each other, if at all. Inspired by [Bunnie Huang's writeups][bunnie_on_biology_reveng] on (computational) biology as a living example of a cross-domain polymath, I'll attempt to write down some thoughts and **pointers on how radare could be used (or not) in bioinformatics** and hopefully manage expectations on what's possible today.

For starters, back in 2015, the simple [Illumina BCL file format][illumina_bcl_format] got included in [radare-extras][radare_extras]. As I was providing some specs and explaning [how DNA sequencing worked][dna_sequencing] in general, [pancake quickly put together a radare plugin][radare_bcl_plugin] for this fairly straightforward file format. 

![BCL format](/blog/images/bcl_format_tldr_csr4.jpg)

Then fast forward into 2018, [radare seems to want more][radare4bio]. Here comes the crux of the matter:

> **Since DNA and ASM are just code running on different architectures, we can reuse the same radare2 reversing workflow right? RIGHT?**

Are they?

Before jumping into sleepless nights of unstoppable implementation, please take the time to read two fun papers touching both domains of reverse engineering and biology. They **reveal how different (or similar) electronics manufacturing and biology can be**.

As stated in [biologists trying to figure out a radio][biologists_repairing_radios] paper:

> "(...) the commonality of the language allows engineers to identify
familiar patterns or modules (a trigger, an amplifier, etc.) in a
diagram of an unfamiliar device"

Food for thought: **how "unfamiliar" of a "device" is biology itself** versus human-**manufactured** consumer wares?

In another, more recent, neuroscinece paper, ["Could a Neuroscientist Understand a Microprocessor?"][neuroscience_understand_ic], some insights come up:

> Much has been written about the differences between computation in silico and computation in vivo (...) the stochasticity, redundancy, and robustness present in biological systems seems dramatically different from that of a microprocessor. But there are many parallels we can draw between the two types of systems.

Bottom line is, they are definitely different yet similar in **some** instances. Without getting overwhelmed by the huge, sometimes messy, amount of domain-specific knowledge to digest, **how can we score "quick wins" for radare2** if an implementation under radare-extras starts to happen? 

Observing bioinformatics [from the RSE perspective][RSE], I see great contributions that could happen in three areas:

1. EBA: Exploratory Bioinformatics Analysis.
2. Scientific algorithm optimization and security.
3. Outreach.

## Radare2 TL;DR for bioinformaticians

Here's some explanation on [how radare works from a user perspective][radare2book]. If you barely recall what assembly was from school, [I'll leave you in good hands][azeria_labs] to catch up with [ARM assembly here][arm_assembly]

Also, some relatively recent UI eyecandy from [Cutter][cutter]:

![Cutter radare2][cutter_screenshot]

Often touted as "steep learning curve" framework due to its commands, radare2 has been misunderstood for years, since in reality, keybindings allow for distraction-free **fast iteration** during binary analysis.


## Bioinfomatics TL;DR for radare2 developers

If you are a r2 developer, those are the formats radare would need to understand and implement to be minimally interesting for our biologist neighbors (optional ones, inside parenthesis):

> [SAM and BAM][sam_spec], ([FASTA, FASTQ, VCF][bioinfo_formats], [CRAM][CRAM], [Crumble][crumble_compression_format])

Now, one could go **the hardcore pancake/Feynman [(brentp?)][brentp] way** and implement file parsers from scratch **or** use some third party library such as [htslib][htslib].

After basic read/write functionality is in place, I think that a potential first win would be to have the ["Midnight Commander"][MC]-equivalent of radare4bio for curious and impatient bioinformaticians.

There's great educational potential if this is implemented right since **radare allows for fast VIM-like iteration and speed during complex analysis**.

For instance, being able to **examine individual reads with VIM shortcuts**, flip/cycle [CIGAR encodings][CIGAR], like with the radare2 bit editor:

![radare2 bit editor](/blog/images/r2_bit_editor.png)

Group reads by some arbitrary criteria, subsample, filter them, write out, etc...

**That is, FAST exploratory bioinformatics analysis (EBA) without the overhead of writing discrete commands or putting together [workflows][commonwl], [pipelines][bcbio] and/or lengthy documentation**


## Outro

1. How would radare **really help** with "biology reverse engineering"?

1. How can radare absorb those "extras" [without introducing a vast dependency tree of bioinfo software][bioconda]? Perhaps a **clean-room implementation** is still of interest nowadays?

1. Would all that coding effort be worth it?

Those are open questions at the time of writing this, but here are some opportunities:

When bioinformaticists analyze data (and are not waiting for big computations to complete), **it is hugely helpful to iterate fast on a particular question**. Keeping focus on the task at hand while answering questions fast and accurately is immensely valuable.

Radare2 is well positioned in this regard, allowing for fast adhoc analysis for the reasons stated before (VIM-like blazing speed shortcuts, focus on speed).

As [a former colleague pointed out][mussolbio], Bioformatics ([and scientific software in general][scisoftware]) is in dire **need for optimization and good software engineering at several levels**: Storage, data processing, security ([read this!][dna_security]), good software design patterns, etc...

**If all else fails, the outreach value of [getting reverse engineers poking into computational biology][sequencer_hacking] is in itself, a huge win, IMHO.**

If you are still reading this, I'm honored :) 

Please ping me physically [during the radare2 2018 con][radarecon2018] or via [twitter `@braincode`](http://twitter.com/braincode) if you want to have a chat about this and other random braindumplings.

  [radarecon2018]: https://rada.re/con/2018/
  [radare4bio]: https://github.com/radare/radare2-extras/issues/165
  [radare_bcl_plugin]: https://github.com/radare/radare2-extras/blob/master/bcl/core_bcl.c
  [biologists_repairing_radios]: https://www.cell.com/cancer-cell/abstract/S1535-6108(02)00133-2?code=cell-site
  [neuroscience_understand_ic]: https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1005268
  [tsp_radar_module_teardown]: https://www.youtube.com/watch?v=5vqSX40seqA
  [eevblog_spectrum_analyzer]: https://www.youtube.com/watch?v=fvTfBwRzpdo
  [dna_security]: https://dnasec.cs.washington.edu/dnasec.pdf
  [bunnie_on_biology_reveng]: https://www.bunniestudios.com/blog/?cat=16
  [sequencer_hacking]: https://www.nature.com/articles/d41586-018-05769-8
  [reddit_reveng]: https://www.reddit.com/r/reverseengineering
  [dna_sequencing]: https://en.wikipedia.org/wiki/DNA_sequencing
  [illumina_bcl_format]: https://www.illumina.com/informatics/sequencing-data-analysis/sequence-file-formats.html
  [crumble_compression_format]: https://academic.oup.com/bioinformatics/advance-article/doi/10.1093/bioinformatics/bty608/5051198
  [plyranges]: https://github.com/sa-lee/plyranges
  [radare_extras]: https://github.com/radare/radare2-extras
  [htslib]: http://www.htslib.org/
  [MC]: https://en.wikipedia.org/wiki/Midnight_Commander
  [sam_spec]: https://samtools.github.io/hts-specs/SAMv1.pdf
  [biostars]: http://biostars.org/
  [CIGAR]: https://wiki.bits.vib.be/index.php/CIGAR
  [bioinfo_formats]: https://bioinformatics-workbook.readthedocs.io/en/latest/introduction/fileFormats/
  [CRAM]: http://www.internationalgenome.org/faq/what-are-cram-files
  [brentp]: https://github.com/brentp
  [RSE]: https://rse.ac.uk/
  [radare2book]: https://radare.gitbooks.io/radare2book/content/
  [r2_cheatsheet]: https://twitter.com/angealbertini/status/685150558915833856
  [cutter_screenshot]: https://raw.githubusercontent.com/radareorg/cutter/master/docs/screenshot.png
  [cutter]: https://github.com/radareorg/cutter
  [assembly]: https://en.wikipedia.org/wiki/Assembly_language
  [arm_assembly]: https://azeria-labs.com/writing-arm-assembly-part-1/
  [azeria_labs]: https://twitter.com/azeria_labs
  [bcbio]: https://github.com/bcbio/bcbio-nextgen
  [commonwl]: https://www.commonwl.org/
  [bioconda]: https://bioconda.github.io/
  [mussolbio]: http://mussol.org/2016/06/11/changing-career-paths/
  [scisoftware]: https://www.nature.com/news/2010/101013/full/467775a.html
  [precision_medicine]: https://en.wikipedia.org/wiki/Precision_medicine

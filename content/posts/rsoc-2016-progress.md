+++
date = "2016-08-02T13:00:10+02:00"
draft = false
title = "RSoC 2016 progress"
slug = "RSoC 2016 progress"
aliases = [
	"RSoC 2016 progress"
]
+++

This year we're hosting our own **Radare Summer of Code**, again!

This is why we have selected 4 students:

 - Aneesh Dogra (FAT PE binaries)
 - Alexandru Razvan Caciulescu (ROP generator)
 - Rakholia Jenish (Kernel level interfaces)
 - Pankaj Kataria (SROP and COOP generators)

### FAT PE binaries
At first, Aneesh Dogra adding the support of FAT PE binaries: those PEs can contain multiple programs inside, like 16bit MZ stub, both native and MSIL/CIL(.NET) code. This task also includes adding support for .NET in RBin, to be able to list the symbols, get the entrypoint, code metadata, etc.

#### Progress so far

*r2* used to crash when loading a fat binary with a lot of sub-bins. Every sub-binary was loaded on RAM at all times and it costed a lot of memory and computation. We tested it using a dyld cache (around 650mb in size) with around 950 libs, and it costed more than 16GBs of ram to load that binary. Now, we can load the same in around 1.2 GBs. We did this by caching the libraries in sdb while parsing the fatbin and then only loading them on ram if we require. Currently I am working on a PE RBinXtr plugin, that would allow PE files to be loaded as fat binaries, hence allowing us to play around with its sub-bins (W16, W32, .NET) in r2.

### Kernel support for r2

Also, Rakholiya Jenish will work on the integration of radare2 with kernel-level interfaces. This includes multiple fixes/features such as:

* Adding support for huge elf files such as Linux kernels with debug information
* Ability to read and parse `/proc/kcore` files
* Linux and Windows kernel drivers to read/write in kernel address space
* Import structures from dwarf and make them available for `pf`
* Multiple scripts to extract information from kernel structures

#### Progress so far

*r2* can now also support working with files larger than RAM. If memory allocation fails while loading a large file into memory, r2 will instead allocate a buffer (128MB as default size) and analysis will be done by re-reading the buffer everytime the physical address is not in the buffer range. The buffer size can also be manually defined using the `R_IO_MAX_ALLOC` environment variable. The test suite cannot test this at the moment, so a future direction could be to look into integrating the manual tests that I'm doing into the test suite.

Currently, I am working on extracting structures in different formats from `.dwarf_info` section so that they can be made available for use with `pf`. We currently plan to use external library `libdwarf`.

### ROP generator

Then, Alexandru Razvan Caciulescu working on the ROPchain generator, based on ragg2. The new tool (or just improved ragg2) will provide parity to well known ROPgadget tool. Also improving the current ROP search algorithms, for example caching rop gadgets in SDB, for quicker retrieval. Something like this syntax is going to be used:
```
register reg1 = 0; 
register reg2 = whatever;
register reg3 = reg1 + reg2;
system(echo reg3);
```

#### Progress so far

For the midterm I focused on the gadget caching in SDB for faster retrieval + lookup while also trying to classify them based on a semantic analysis.

From now on all the ```/R``` commands which previously only searched and displayed gadgets also cache them in the SDB. We decided to store them as <start_addr>:<length> For querying them we have the new commands:

* ```/Rk``` display all gadgets
* ```/Rkj``` display in JSON format
* ```/Rkq``` display only the offsets

The ```Ps``` and ```Po``` commands have also been enhanced to save/load the ROP SDB into the projects.

Currently I am working on the semantic classification of gadgets. Each gadget will be classified in on of the categories presented here: https://hackmd.io/s/By-S2GLu
For this task the radare framework is ideal because we can emulate each assembly instruction and see which REGs, FLAGs and MEM addr are accessed and how they are changed, thus providing all the information required for assigning a class to each gadget. The main challenge for this was actually understanding how this works and how radare handles all of this in the background so I can adapt it to my specific needs for this task.


### SROP/COOP generator

Pankaj Kataria will add the ability to generate SROP and COOP (Counterfeit Object Oriented Programming) sequences to the same tool, sharing the code/infrastructure with ROP generation code.

Counterfeit Object-oriented programming is an advanced and fairly new technique which even by hand is hard and error prone process so, more importance is given on the design for the tool automating this technique is make is less painfull to use. Currently i am implementing Searcher for finding vfgadgets (virtual functions) used by the input binary.

##### Situtation

The problem of finding all the virtual function boils down to find all vtable's being used in a binary. But as the exact location of the all the vtable's is not known so, to solve this I am using an algorithm made by combination techniques described in this and this paper. The abstract algorithm composed is described as follows :   

* Start by analyzing the data segment and scanning each location in it. 
* If current location is referenced from a code segment and its value  is a pointer to a function, then mark it as a start of vtable.
* Else continue to search.
* Other elements of the virtual table must be must be unreferenced pointers to functions, so finding the size of virtual table is straightforward as Vtable ends with the first locations that is either referenced from the program code, orht is not a pointer to a function. 

Once the virtual function table is identified Runtime Type information(RTTI) can be recovered as well because in most of Application Binary Interface pointer to RTTI structure of a class always precedes the corresponding virtual table. Later using RTTI polymorphic class hierarchies is reconstructed exactly as it was in the source C++ program. 

---------

Please, help us to pay the proper amount of money to our students by supporting our crowdfunding campaign at IndieGoGo: https://igg.me/at/rsoc2016

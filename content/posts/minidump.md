+++
date = "2017-10-02T08:00:00+02:00"
draft = false
title = "Using r2 to analyse Minidumps"
slug = "minidump"
aliases = [
	"Minidump"
]
+++

The minidump format is used by Microsoft for storing user-mode memory dumps. It is an openly documented format that is also extensible, but it is almost always analysed in WinDbg [\[1\]](https://msdn.microsoft.com/en-us/library/windows/desktop/ms680369(v=vs.85).aspx)[\[2\]](https://msdn.microsoft.com/en-us/library/windows/desktop/ms680378(v=vs.85).aspx). 

This article describes how to perform analysis of minidumps using radare2 **mdmp** module.

## Installation

If you use radare2 from git as recommended nothing to do, you should already be able to identify the fileformat as `mdmp` rather than `any`.

# The format

The format is inherently simple as it is a collection of streams that vary depending on their content.

```
$ r2 mini.dmp
[0x00690efa]> i
blksz    0x0
block    0x100
fd       6
file     mini.dmp
format   mdmp
iorw     false
mode     -r--
size     0xc53a7be
humansz  197.2M
type     MDMP (MiniDump crash report data)
arch     x86
binsz    206809022
bintype  mdmp
bits     32
canary   false
crypto   false
endian   little
flags    0x00061826
havecode true
hdr.csum 0x00000000
linenum  false
lsyms    false
machine  i386
maxopsz  16
minopsz  1
nx       false
os       Windows NT Workstation 6.1.7601
pcalign  0
pic      false
relocs   false
rpath    NONE
static   false
streams  13
stripped false
va       true
```

Above, we can see that the file `mini.dmp` contains 13 stream and is of the file type MiniDuMP (MDMP). For additional information on the basics of the MDMP format you can refer to a great [blog post](http://moyix.blogspot.co.uk/2008/05/parsing-windows-minidumps.html) by Brendan Dolan-Gavitt.


## Parsing

The MDMP format results in an information rich file, thus simple parsing of the file is sometimes sufficient. This is easy to do with radare2's `pf` command see [parsing a fileformat with radare2 article](http://radare.today/posts/parsing-a-fileformat-with-radare2/). 

Below is a list of the currently supported structures:

```
[0x00000000]> pf.
pf.mdmp_memory_descriptor       pf.mdmp_misc_info               pf.mdmp_thread_info             
pf.mdmp_thread_list             pf.mdmp_module                  pf.mdmp_unloaded_module_list    
pf.mdmp_system_info             pf.mdmp_module_list             pf.mdmp_thread_info_list        
pf.mdmp_location_descriptor     pf.mdmp_string                  pf.mdmp_memory64_list           
pf.mdmp_vs_fixedfileinfo        pf.mdmp_location_descriptor64   pf.mdmp_header                  
pf.mdmp_unloaded_module         pf.mdmp_memory_descriptor64     pf.mdmp_directory               
pf.mdmp_memory_info             pf.mdmp_thread                  pf.mdmp_handle_data_stream
pf.mdmp_memory_info_list
```

By opening the file in radare2's predefined binary structures mode it is possible to view the structures of the MDMP file. The `mini.dmp` file from above has the header depicted below: 

```
[0x00000000]> pf.mdmp_header
          Signature : 0x00000000 = MDMP
            Version : 0x00000004 = 1812113299
    NumberOfStreams : 0x00000008 = 13
 StreamDirectoryRVA : 0x0000000c = 32
           CheckSum : 0x00000010 = 0
      TimeDateStamp : 0x00000014 = Tue Jan 31 18:44:24 2017
              Flags : 0x00000018 =  Flags (bitfield) = 0x00061826 : MiniDumpWithFullMemory | MiniDumpWithHandleData | MiniDumpWithUnloadedModule | MiniDumpWithFullMemoryInfo | MiniDumpWithThreadInfo | MiniDumpIgnoreInaccessibleMemory | MiniDumpWithTokenInformation
```

It should be noted that the flags above portray the information that will be written to the MDMP file. Therefore, certain flags are required to perform different types of analysis. 
An astute reader will notice that the `NumberOfStreams` matches that reported by the `file` binary. Using the `NumberOfStreams` and `StreamDirectoryRVA` it is possible to print out the list of directory structures. 

Below are the structures for the thread information and module list describing their location and size:

```
[0x00000000]> pf.mdmp_stream_directory [13]? (mdmp_directory)directories
[0x00000000]> pf.mdmp_stream_directory @ 32
 directories :
[
[snip]
                struct<mdmp_directory>
            StreamType : 0x0000002c = StreamType (enum mdmp_stream_type) = 0x11 ; ThreadInfoListStream
              Location :
                      struct<mdmp_location_descriptor>
                   DataSize : 0x00000030 = 1676
                        RVA : 0x00000034 = 1728
                struct<mdmp_directory>
            StreamType : 0x00000038 = StreamType (enum mdmp_stream_type) = 0x4 ; ModuleListStream
              Location :
                      struct<mdmp_location_descriptor>
                   DataSize : 0x0000003c = 21820
                        RVA : 0x00000040 = 3404
[snip]
```

The module list contains the information about loaded binaries and libraries. For this process dump there are 202 modules, it is most likely that the first one is the executable while the rest are libraries.

```
 NumberOfModule : 0x00000d4c = 202
        Modules :
[
                struct<mdmp_module>
                 BaseOfImage : 0x00000d50 = (qword)0x0000000000660000
                 SizeOfImage : 0x00000d58 = 2625536
                    CheckSum : 0x00000d5c = 2648680
               TimeDateStamp : 0x00000d60 = Sat Nov 20 09:37:55 2010
               ModuleNameRVA : 0x00000d64 = 28646
                 VersionInfo :
                      struct<mdmp_vs_fixedfileinfo>
                                              dwSignature : 0x00000d68 = 4277077181
                                           dwStrucVersion : 0x00000d6c = 65536
                                          dwFileVersionMs : 0x00000d70 = 393217
                                          dwFileVersionLs : 0x00000d74 = 498156650
                                       dwProductVersionMs : 0x00000d78 = 393217
                                       dwProductVersionLs : 0x00000d7c = 498156650
                                          dwFileFlagsMask : 0x00000d80 = 63
                                              dwFileFlags : 0x00000d84 = 0
                                                 dwFileOs : 0x00000d88 = 262148
                                               dwFileType : 0x00000d8c = 1
                                            dwFileSubtype : 0x00000d90 = 0
                                             dwFileDateMs : 0x00000d94 = 0
                                             dwFileDateLs : 0x00000d98 = 0
                    CvRecord :
                      struct<mdmp_location_descriptor>
                   DataSize : 0x00000d9c = 37
                        RVA : 0x00000da0 = 64538
                  MiscRecord :
                      struct<mdmp_location_descriptor>
                   DataSize : 0x00000da4 = 0
                        RVA : 0x00000da8 = 0
                   Reserved0 : 0x00000dac = (qword)0x0000000000000000
                   Reserved1 : 0x00000db4 = (qword)0x0000000000000000
[snip]
```

These are the basics of structure traversal, and will hopefully show how easy it is to manually parse the MDMP format.

## Analysis

### EAT Hooking

The MDMP plugin parses the streams into information that can then be presented within radare2. While this is beneficial, parsing the data sections can be slow, therefore loading times can be decreased with the `-z` parameter, which prevents the loading of strings. 

The in-depth parsing of the MDMP format can result in an extensive amount of information being displayed when using radare2's `i` commands (`~??` is your friend!).

The best way to demonstrate the capability of this plugin is by example. The process dump `mini.dmp` is that of `explorer.exe` that was originally obtained due to some suspicious unknown module hooks. The sections list contains all of the mapped executables and DLLs, with a simple grep, it is possible to confirm that the dump is indeed for `explorer.exe`:

```
[0x00690efa]> iSq~exe
0x660000 0x8e1000 m---- C:_Windows_explorer.exe
```

Furthermore the layouts for each module are also displayed, below is that for `explorer.exe`:

```
[0x00690efa]> iSq
[snip]
0x660000 0x8e1000 m---- C:_Windows_explorer.exe
0x661000 0x710600 m-r-x .text
0x711000 0x713c00 m-rw- .data
0x714000 0x8d7000 m-r-- .rsrc
0x8d7000 0x8e0400 m-r-- .reloc
[snip]
```

One of the hooked functions in question is `InternetReadFile` of hook type EAT (Export Address Table). Thus the function should be visible in the export table for the library that has been hooked. A quick grep through the exports reveals the DLL:

```
[0x00690efa]> iE~InternetReadFile
vaddr=0x757bfa90 paddr=0x0ac8b24e ord=176 fwd=NONE sz=0 bind=GLOBAL type=FUNC name=WININET.dll_InternetReadFile
```

We can see the function immediately jumping off to an address in a low page memory region:

```
[0x00690efa]> pd 10 @ 0x757bfa90
        └─< ;-- WININET.dll_InternetReadFile:
        └─< 0x757bfa90      e99a099b8e     jmp 0x417042f
            0x757bfa95      83ec38         sub esp, 0x38               ; '8'
            0x757bfa98      56             push esi
            0x757bfa99      6a38           push 0x38                   ; '8' ; '8'
            0x757bfa9b      8d45c8         lea eax, [ebp - 0x38]
            0x757bfa9e      6a00           push 0
            0x757bfaa0      50             push eax
            0x757bfaa1      e88c2c0200     call 0x757e2732
            0x757bfaa6      83c40c         add esp, 0xc
            0x757bfaa9      8d45c8         lea eax, [ebp - 0x38]
```

By seeking to the `jmp` it is possible to check the legitimacy of the memory section:

```
[0x0416fbcf]> s 0x417042f
[0x0417042f]> S.
0x069787be 0x069797be Memory_Section_203
```

Clearly this section does not reside in a valid DLL, but does it contain valid assembly?

```
[0x0417042f]> pd 20
            0x0417042f      51             push ecx
            0x04170430      9c             pushfd
            0x04170431      b9e4131704     mov ecx, 0x41713e4
            0x04170436      f60101         test byte [ecx], 1          ; [0x1:1]=68
        ┌─< 0x04170439      7531           jne 0x417046c
        │   0x0417043b      55             push ebp
        │   0x0417043c      57             push edi
        │   0x0417043d      56             push esi
        │   0x0417043e      52             push edx
        │   0x0417043f      53             push ebx
        │   0x04170440      50             push eax
        │   0x04170441      64ff35000000.  push dword fs:[0]
        │   0x04170448      64ff35340000.  push dword fs:[0x34]
        │   0x0417044f      51             push ecx
        │   0x04170450      e8abfbf2ff     call 0x40a0000
        │   0x04170455      83c404         add esp, 4
        │   0x04170458      648f05340000.  pop dword fs:[0x34]
        │   0x0417045f      648f05000000.  pop dword fs:[0]
        │   0x04170466      58             pop eax
        │   0x04170467      5b             pop ebx
```

Yes. Furthermore, there is an immediate value that is moved to `ecx` and then pushed to the stack as the first argument for the function call `0x40a0000`. A few variables look like they could be of interest (memory addresses?) at the offset stored in `ecx`.

```
[0x0417042f]> px 64 @ 0x041713e4
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x041713e4  0004 0000 0000 0a04 1810 0a04 0000 0000  ................
0x041713f4  0000 0000 0000 0000 0000 0000 0000 0000  ................
0x04171404  5027 a466 0000 0000 0000 0000 90fa 7b75  P'.f..........{u
0x04171414  2f04 1704 0000 0000 8bff 558b ec83 ec38  /.........U....8
```

In the function `0x40a0000` it eventually access one of these potential addresses, the one at offset 0x20.

```
[0x0417042f]> pd 80 @ 0x40a0000
            ;-- section_end.Memory_Section_191:
            ;-- section.Memory_Section_192:
            0x040a0000      55             push ebp                    ; section 192 va=0x040a0000 pa=0x027c37be sz=4096 vsz=4096 rwx=m-r-x Memory_Section_192
            0x040a0001      8bec           mov ebp, esp
            0x040a0003      83ec48         sub esp, 0x48               ; 'H'
            0x040a0006      8b4508         mov eax, dword [ebp + 8]    ; [0x8:4]=13 ; "\r"
[snip]
       ││   0x040a00a3      8b4508         mov eax, dword [ebp + 8]    ; [0x8:4]=13 ; "\r"
       ││   0x040a00a6      8b4820         mov ecx, dword [eax + 0x20] ; [0x20:4]=3
       ││   0x040a00a9      ffd1           call ecx
```

Below is the address held for that offset.

```
[0x0417042f]> pxw 4 @ 0x041713e4+0x20
0x04171404  0x66a42750                                P'.f
```

By dropping the 2 least significant bytes it is possible to do a fuzzy search in the sections list:

```
[0x00690efa]> iSq~0x66a4
0x66a40000 0x66a41000 m-r-- Memory_Section_506
0x66a41000 0x66a45000 m-r-x Memory_Section_507
0x66a45000 0x66a46000 m-r-- Memory_Section_508
0x66a46000 0x66a47000 m-rw- Memory_Section_509
0x66a47000 0x66a49000 m-r-- Memory_Section_510
0x66a40000 0x66a49000 m---- C:_Program_Files_McAfee_Endpoint_Security_Threat_Prevention_IPS_EpMPThe.dll
0x66a41000 0x66a44400 m-r-x .text_141
0x66a45000 0x66a45e00 m-r-- .rdata_12
0x66a46000 0x66a46200 m-rw- .data_140
0x66a47000 0x66a47400 m-r-- .rsrc_147
0x66a48000 0x66a48400 m-r-- .reloc_141
```

Voila, the offending DLL - `EpMPThe.dll` which is part of the McAfee Endpoint Security Suite. Did you know that AV also uses malware-like techniques ;)

### EXE Carving

Carving binaries from a memory dump is an incredibly useful technique, especially when it comes to injected malware. Fortunately r2 makes very light work of such a technique [[3]](http://radare.today/posts/carving-bins/). To demonstrate this, a memory dump of `hello.exe` has been created and will be used for this example. For the sake of simplicity, we are going to carve this executable out of its MDMP file. This sample is part of the MDMP regression test suite and is available [here](https://github.com/radare/radare2-regressions/blob/master/bins/mdmp/hello64.dmp), for those who wish to reproduce.

```
$ r2 hello64.dmp -z
[0x7ff7aebd0f04]> iS~exe
idx=53 vaddr=0x00400000 paddr=0x00092c8f sz=151552 vsz=151552 perm=m---- name=C:_Users_Developer_Desktop_hello64.exe
```

Using the data above it is possible to carve `hello64.exe` from the MDMP file. This is achieved by seeking to the start address of section and dumping it.

```
[0x00401500]> s 0x00400000
[0x00400000]> px @ 0x00400000
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x00400000  4d5a 9000 0300 0000 0400 0000 ffff 0000  MZ..............
0x00400010  b800 0000 0000 0000 4000 0000 0000 0000  ........@.......
0x00400020  0000 0000 0000 0000 0000 0000 0000 0000  ................
0x00400030  0000 0000 0000 0000 0000 0000 8000 0000  ................
0x00400040  0e1f ba0e 00b4 09cd 21b8 014c cd21 5468  ........!..L.!Th
0x00400050  6973 2070 726f 6772 616d 2063 616e 6e6f  is program canno
0x00400060  7420 6265 2072 756e 2069 6e20 444f 5320  t be run in DOS
0x00400070  6d6f 6465 2e0d 0d0a 2400 0000 0000 0000  mode....$.......
0x00400080  5045 0000 6486 1200 b79c 2058 00aa 0100  PE..d..... X....
0x00400090  e605 0000 f000 2700 0b02 0219 001e 0000  ......'.........
0x004000a0  0044 0000 000c 0000 0015 0000 0010 0000  .D..............
0x004000b0  0000 4000 0000 0000 0010 0000 0002 0000  ..@.............
0x004000c0  0400 0000 0000 0000 0500 0200 0000 0000  ................
0x004000d0  0050 0200 0006 0000 1946 0200 0300 0000  .P.......F......
0x004000e0  0000 2000 0000 0000 0010 0000 0000 0000  .. .............
0x004000f0  0000 1000 0000 0000 0010 0000 0000 0000  ................
[0x00400000]> wtf hello64.exe 151552
dumped 0x429000 bytes
Dumped 151552 bytes from 0x00400000 into hello64.exe
```

That is all there is to carving the binary from a memory dump, but in order to analyse the binary in r2 under isolation a little extra work is required. To be more precise, the section headers need patching to update their PointerToRawData and SizeOfRawData to that of the VirtualAddress and VirtualSize, respectively.

```
struct IMAGE_SECTION_HEADER 
{
  char  Name[IMAGE_SIZEOF_SHORT_NAME];
  union {
    long PhysicalAddress;
    long VirtualSize;
  } Misc;
  long  VirtualAddress;
  long  SizeOfRawData;
  long  PointerToRawData;
  long  PointerToRelocations;
  long  PointerToLinenumbers;
  short NumberOfRelocations;
  short NumberOfLinenumbers;
  long  Characteristics;
}
```

By making use of `r2pipe` we can script this task. Below is the `patch` function taken from a bin carver script that demonstrates the steps required to patch the PE's section headers [[4]](https://github.com/countercept/radare2-scripts/blob/master/r2_bin_carver.py):

```python
def patch(file_path):
    r2 = r2pipe.open(file_path)
    info = r2.cmdj('ij')
    if info['bin']['bintype'] != 'pe':
        print("[+] Patching not possible, only PE files supported!")
        exit()
    r2 = r2pipe.open(file_path, ['-w', '-nn'])

    print('[+] Patching...')
    # TODO: The pf structs don't exist at the time of writing! And pfsj does
    # not exist. For these reasons lets just seek and be lazy
    # pf [8]zxxxxxxwwx
    e_elfnew_addr = r2.cmdj('pfj x @ 0x3c')[0]['value']
    numberOfSections = r2.cmdj('pfj w @ %i' % (e_elfnew_addr + 0x6))[0]['value']
    sizeOfOptionalHeader = r2.cmdj('pfj w @ %i' % (e_elfnew_addr + 0x14))[0]['value']
    base_addr = e_elfnew_addr + 24 + sizeOfOptionalHeader

    print('[+] Found %i sections to patch' % (numberOfSections))
    for i in range(0, numberOfSections):
        addr = base_addr + 40 * i
        print('[+] Patching Section %i.' % (i))
        VirtualSize = r2.cmdj('pfj x @ %i' % (addr + 0x08))[0]['value']
        print('\tSetting VirtualSize to 0x%x' % (VirtualSize))
        r2.cmd('wv %i @ %i' % (VirtualSize, addr + 0x10))
        VirtualAddress = r2.cmdj('pfj x @ %i' % (addr + 0x0c))[0]['value']
        print('\tSetting PointerToRawData to 0x%x' % (VirtualAddress))
        r2.cmd('wv %i @ %i' % (VirtualAddress, addr + 0x14))

    print('[+] Patching done')
```

Putting the above into practice, it is now possible to carve and patch the binary in a single line.

```
$ ./r2_bin_carver.py hello64.dmp 0x00400000 0xe25000 -b MZ -p
[+] Checking for magic: MZ - 4d5a
[+] Magic found, carving...
[+] Carving to hello64.dmp.0x00400000
[+] Patching...
[+] Found 18 sections to patch
[+] Patching Section 0.
        Setting VirtualSize to 0x1d38
        Setting PointerToRawData to 0x1000
[+] Patching Section 1.
        Setting VirtualSize to 0x98
        Setting PointerToRawData to 0x3000
[snip]
[+] Patching Section 16.
        Setting VirtualSize to 0x2c70
        Setting PointerToRawData to 0x21000
[+] Patching Section 17.
        Setting VirtualSize to 0x4f0
        Setting PointerToRawData to 0x24000
[+] Patching done
```

With the patching complete it is now possible to analyse the binary.

## Conclusion

Hopefully this blog post has shown you some radare2 tips for analysing memory dumps. Stay tuned as we’re still working on improving minidump support, so stay tuned for more awesomeness!

-- Alex Kornitzer ([@AlexKornitzer](https://twitter.com/alexkornitzer))

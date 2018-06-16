+++
date = "2018-06-16T01:45:16+02:00"
draft = false
title = "Android Crackme and Structure offset propagation"
slug = "crackme_with_tl"
aliases = [
	"crackme with tl"
]
+++

Today we will look into the recently introduced feature in r2 (so be warned it's still a WIP xD) which is Structure offset propagation and use this to solve a crackme based on reversing an Android JNI (Java Native Interface) library.

This challenge is originally from NDH2012-wargame, So we are provided with an [NDH.apk](https://github.com/sivaramaaa/CTF_repo/blob/master/NDH2k12/NDH.apk) file, Now after decompiling and using JD-GUI to browse through the code we can find some interesting functions : 

```java 
// Java source file
static
{
     System.loadLibrary("verifyPass");
}
NDHActivity localNDHActivity = NDHActivity.this;
String str1 = this.val$tv2.getText().toString();
String str2 = localNDHActivity.print(str1);
```
After reading through the code we can find that the validation routine is contained in a [JNI lib](https://github.com/sivaramaaa/CTF_repo/blob/master/NDH2k12/libverifyPass.so) file.

```
$ file libverifyPass.so
libverifyPass.so: ELF 32-bit LSB shared object, ARM, EABI5 version 1 (SYSV), dynamically linked, stripped
```

So let's start by opening it in r2 (~~IDA~~ :P) -

```
$ r2 ./libverifyPass.so
[0x00000f50]> aaa
...
[0x00000f50]> afl
0x00000f38    1 12           sym.imp.__gnu_Unwind_Find_exidx
0x00000f44    1 12           sym.imp.difftime
0x00000f50    1 12           entry0
0x00000f60   15 500          sym.Java_com_app_ndh_NDHActivity_print
...
```
We could easily spot the "sym.(..).print" function that was called from the java source file 

```
[0x00000f60]> pdf
|   sym.Java_com_app_ndh_NDHActivity_print ();
| ; var int local_4h @ sp+0x4
  ....
| 0x00000f60      30b5     push {r4, r5, lr}
| 0x00000f62      cdb0     sub sp, 0x134
| 0x00000f64      7e4c     ldr r4, [0x00001160]
| 0x00000f66      7c44     add r4, pc
| 0x00000f68      0390     str r0, [sp + local_ch] //
| 0x00000f6a      0291     str r1, [sp + local_8h] // Arguments stored in stack
| 0x00000f6c      0192     str r2, [sp + local_4h] //
  ....
| 0x00000f76      039b     ldr r3, [sp + local_ch]
| 0x00000f78      1a68     ldr r2, [r3] // JNIEnv struct loaded
| 0x00000f7a      a923     movs r3, 0xa9
| 0x00000f7c      9b00     lsls r3, r3, 2
| 0x00000f7e      d358     ldr r3, [r2, r3]
 ....
| 0x00000f8a      9847     blx r3
```
We could see that one of the arguments passed to this print function is a **JNIEnv** struct pointer which contains info about many callback function (For detailed explaination about JNI , take a look at this [wiki](https://en.wikipedia.org/wiki/Java_Native_Interface))

Thia program uses this type of call 3 times, with the indexes 0x2A4, 0x290 and 0x29C. 

To retreive the names of the corresponding functions, we can directly use the [jni.h](https://github.com/sivaramaaa/CTF_repo/blob/master/NDH2k12/jniapi.h) file or this [doc](https://docs.oracle.com/javase/1.5.0/docs/guide/jni/spec/functions.html#wp23720) which contains the indexes in the JNIEnv interface function table : 

```cc
0x24A / 4 = 169 --> const jbyte* GetStringUTFChars(JNIEnv *env, jstring string, jboolean *isCopy);
0x290 / 4 = 164 --> jsize GetStringLength(JNIEnv *env, jstring string);
0x29C / 4 = 167 --> jstring NewStringUTF(JNIEnv *env, const char *bytes);
```

As u can see it's a bit painful process, this is where our struct offset propagation comes into the picture!

```
[0x00000f60]> to ./jni.h (load the header file)
[0x00000f60]> ts 
_JavaVM
JNIInvokeInterface
JNINativeInterface
_JNIEnv
```

You can either use esil emulation or debugging to find out the address of JNIEnv struct and then link the struct to the address and watch the magic happen :) 

```
[0x00000f60]> tl JNINativeInterface = 0x464c457f
[0x00000f60]> pdf
...
| 0x00000f7c      9b00   lsls r3, r3, 2
| 0x00000f7e      d358   ldr r3, [r2, r3] ; JNINativeInterface.GetStringUTFChars
| 0x00000f84      081c   adds r0, r1, 0
| 0x00000f86      111c   adds r1, r2, 0
| 0x00000f88      0022   movs r2, 0
| 0x00000f8a      9847   blx r3
...
| 0x0000100e      d358   ldr r3, [r2, r3] ; JNINativeInterface.GetStringLength
| 0x00001010      0399   ldr r1, [sp + local_ch]
| 0x00001012      019a   ldr r2, [sp + local_4h]
| 0x00001014      081c   adds r0, r1, 0
| 0x00001016      111c   adds r1, r2, 0
| 0x00001018      9847   blx r3
....
| 0x00001110      a723   movs r3, 0xa7
| 0x00001112      9b00   lsls r3, r3, 2
| 0x00001114      d258   ldr r2, [r2, r3] ; JNINativeInterface.NewStringUTF

```

So coming to the solution of this challenge, as we see there are some string length check happening and at last two strings are loaded from .rodata section and xored together to print the flag

```python
>> a = [0x52,0x1A,0x09,0x7B ...]
>> b = [0x5C,0x20,0x72,0x10 ...]
>> "".join( [ chr(a[i] ^ b[i]) for i in range(len(a)) ] )
'Radare2_is_great' (obviously not the flag, try it on ur own to get it :P)
```

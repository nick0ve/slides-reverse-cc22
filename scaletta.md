* Reverse Engineering

  * Why?
    * Most of the software is closed source, but why would you want to reverse engineering a piece of sw?
    * Curiosity
    * Malware analysis: What is this not suspicious antivirus.exe doing?
    * Exploit analysis: Your phone got hacked upon receiving a nice cat image https://googleprojectzero.blogspot.com/2021/12/a-deep-dive-into-nso-zero-click.html, how do you analyze such a thing?
    * Software Preservation: Your company is using a legacy sw from 40 yrs ago, the developer doesn't exist anymore, what do you do? 
    * Compatibility: you want to try a way to make your android phone communicate with a smart thing in your house, but the communicate sw only exists for ios phones
    * Security assessment
      * You buy a surveillance camera from china because it is cheap https://www.wired.it/economia/business/2021/07/28/hikvision-telecamere-cina-sorveglianza-ministero-cultura-lavoratori/ how do you assess if it is safe to put this thing inside critical environments?
    * Vulnerability Research: Riot is willing to pay you up to $100k if you find and exploit a bug in the Vanguard anticheat
    * My hw is mine, I do what I want with it: well yes but actually everything is locked down, you have to bypass the security features before doing anything
    * sbarrato_Game Cheating_sbarrato
      * the enemy position (x,y,z) is stored at some memory address, what happens if you write a program that take that position and do stuffs based on this?

  * How?

    * Static Analysis
      * analyze the program by examing code and data without executing it
        * the good
          * it is the only way to evaluate all the possible execution paths
          * it is harder than observing the program at runtime, but it is a slow approach
          * bypass by default antidebugging checks
        * the bad
          * it is harder to trace what data a function is using, if that data cames from 20 nested calls :)
          * the program is heavily obfuscated, example photo of a simple MBA function
    * Dynamic Analysis
      * analyze the program by observing it during runtime
        * it is faster
        * it is easier to reason about what a piece of code is doing, when you know what data it is using (which might be hard to trace during static analysis)
        * but you can't always rely on dynamic analysis
          * the program is a malware and you are scared of running it without analyzing it
          * the program uses anti debug techniques, which you have to overcome before being able to dynamically analyze it
          * you can't run the program
    * In practice you mix static and dynamic analysis!
  
  * Executable File Formats
    * There are a lot of executable file formats, the common ones are:
      * PE -> Portable Executable -> Windows
      * ELF -> Executable and Linkable Format -> Common on Unix-Like Systems
      * Mach-O -> Mach object file format -> macOS and iOS
    * In this course we will focus on ELF files, which are used by Linux
  
  * ELF File: Executable and Linkable Format
    * It is the common file format to represents executables and shared libraries on Linux
    * ELF Ehdr - Image - Describe by voice - highlight in the slides the useful fields
    * ELF Phdr - Image - Describe by voice - highlight in the slides the useful fields
    * ELF Shdr - Image - Describe by voice - highlight in the slides the useful fields
    * Stripping
      * Some information contained by ELF Ehdr, Phdr and Shdr are not needed by the linux kernel to load and execute a program
      * In fact you can completely remove all the Section headers
      * You can do evil stuffs, like corrupting some fields contained by these headers, linux is not gonna complain but oh boy your favorite reverse engineering tool will leave you alone!
      * This is useful whenever you want to shrink the size of your application before shipping to the final user
      * You can use the command `strip` as a cheap way to make reverse engineering of your application harder, because it strips away all the function names

  * Static Analysis
    * Tools
      * Initial recoinnasance
        * strings 
        * https://github.com/mandiant/flare-floss strings on steroids
        * ldd
        * nm -n
        * readelf
      * ghidra - binja - ida - hopper - r2
        * It depends on how rich you are, ghidra is $free, binja costs 80$ for students, ida 10k$ yearly, hopper idk, r2 costs your mental health because it segfaults frequently
    * what are you telling me ghidra? https://byte.how/posts/what-are-you-telling-me-ghidra/ TODO ESTRARRE INTO SLIDE
    * tips n tricks
      * Focus on the data flow, every reverse engineering tool does this analysis, DFA
        * example of memcpy, strcpy, memcmp, strcmp
      * Black box analysis for some functions too hard to analyze
        * <screen of a mba obfuscated function with one input>
          Try to call the funciton with different inputs and guess the behaviour based on the outputs
        * Still can't understand what it is doing?
          * Maybe It is used only with few different inputs, in that case you can treat those calls as constants
          * Try to guess what it is doing based on the context, most of the time having a big picture of what is happening is fine enough
  
  * Dynamic Analysis
    * Initial recoinnasance
      * strace - trace syscalls
        * example
        * some useful snippets
      * ltrace - trace library calls
        * example
    * GDB - scripting - scripting with python
      * Essential commands:
        * `n` - next line, stepping **over** function calls (source code granularity)
        * `ni` - next instruction, stepping **over** function calls (assembly granularity)
        * `s` - next line, stepping **into** function calls (source code granularity)
        * `si` - next instruction, stepping **into** function calls (assembly granularity)
        * `fin` - run until selected stack frame returns, that's what you want to call whenever you fatfinger `si` command instead of `ni`
        * `p expr` - display the value of an expression
        * `set var=expr` - evaluate expr and set `var` value to it
          * `set $rax=1`
        * `x/<N><U><F>` - eXamine memory
          * `N` is the repeat count, it specifies how much memory (counting by units `u`) to display
          * `U` is the unit size, cause of confusion because the naming is different from other reverse engineering tools, it can be one of:
            * `b` -> 1 Byte.
            * `h` -> Halfwords (two bytes). // Usually WORD
            * `w` -> Words (four bytes). // Usually DWORD
            * `g` -> Giant words (eight bytes). // Usually QWORD
          * `F` is the display format, it can be one of: (sorted by usefulness, you don't have to remember them all)
            * `i` -> Regard as instructions and displays the corresponding assembly.
            * `x` -> Regard the bits of the value as an integer, and print the integer in hexadecimal.
            * `z` -> Like `x` formatting, the value is treated as an integer and printed as hexadecimal, but leading zeros are printed to pad the value to the size of the integer type.
            * `a` -> Print as an address, both absolute in hexadecimal and as an offset from the nearest preceding symbol.
            * `d` -> Print as integer in signed decimal.
            * `u` -> Print as integer in unsigned decimal.
            * `o` -> Print as integer in octal.
            * `t` -> Print as integer in binary. The letter ‘t’ stands for “two”.
            * `c` -> Regard as an integer and print it as a character constant.
            * `f` Regard the bits of the value as a floating point number and print using typical floating point syntax.
            * `s` Regard as a string
        * Execution control:
          * `return [expr]` - pop selected stack frame without executing [setting return value]
          * `until [location]` - run until next instruction (or location)
          * `jump *address` - resume execution at specified address
        * Display informations:
          * `bt [n]` - print trace of all frames in stack; or of n frames—innermost if n>0, outermost if n<0
          * `frame [n]` - select frame number n or frame at address n, if no n display current frame
          * `info args` - arguments of selected frame
          * `info locals` - local variables of selected frame
          * `info reg [rn]` - show register value
    * hooking
    * LD_PRELOAD
    * frida
    * Unicorn Emulator
      * Create an emulator object
      * Hook at instruction granularity
      * It is a bare metal emulator, aka it doesn't know anything about the environment
      * If your program calls a read() syscall to take input from a filedescriptor, 
        you might want to emulate the corresponding behaviour
    * GhidraEMU 
      * Can emulate every architecture supported by the decompiler
      * Usefulness? 
        * The architecture that you are analyzing is not supported by unicorn
        * You are writing a script to deobfuscate some functions and you want to emulate a piece of code
  
  * (A) Reverse Engineering Methodology
    * Initial recoinnasance
      * poke the program
      * scratch the surface, look around in the functions recognized by your favourite analysis tool 
      * make an idea of what it might be doing under the hood
    * During analysis
      * make hypothesis and test them, it will speed you up
      * if you are reversing a huge program, avoid focus on every detail until it is necessary, it might not be useful
        => zoom in and zoom out as needed
        if you are doing VR and see something suspicious it might be time to dig deeper

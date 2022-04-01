---
title: American Fuzzy Lop is your friend!
description: Usage of popular fuzzer to uncover bugs in embedded libraries and modules
author: djasinski
---

<!-- excerpt start -->

Fuzz testing also knowns as fuzzing is a rather unconventional method of software testing. Instead of providing a finite set of known and well-described test cases with deterministic input, it mutates the next test cases based on the initial passing (not crashing) test case.

In other words - input is mutated (using various methods such as genetic algorithms or classic random approach) and the given program is running in the loop until a crash will occur.

Being known in cybersecurity when it comes to classic computer software development (fuzzing technique helped to uncover bugs and vulnerabilities in plenty of open source projects - e.g Firefox Web Browser, PuTTY, bash, Qt, SQLite...) - it is still rather unknown within embedded engineers community.

This article is going to convince you that the fuzzing technique is actually your friend!

<!-- excerpt end -->

{% include newsletter.html %}

{% include toc.html %}

## What is the fuzzing technique?

In comparision with traditional software testing methodologies fuzzers produce the testing datas on themselves - mostly basing on valid user provided seed. For example, if we are dealing with audio file player - we are providing valid file in specified format (e.g *.wav) - and then fuzzer modifies the file content expecting the program to crash.

According to the personal recollection of [Gerald M. Weinberg’s](http://secretsofconsulting.blogspot.com/2017/02/fuzz-testing-and-fuzz-history.html) - technique similar to which nowadays we are calling fuzzing was used back in the 1950s. He mentions that it was the standard practice to test newly written programs taking punched cards directly from the thrash in search of unexpected behavior. A short note is mentioning that it was so common that it had no name.

The name ‘fuzzing’ was brought by professor [Barton Miller](https://pages.cs.wisc.edu/~bart/fuzz/CS736-Projects-f1988.pdf) when developing a project during advanced operating systems class. The utility was able to automatically generate various random command line parameters and arguments to verify the reliability of UNIX environment utilities.

## Fuzzer software categorization

There are various differences in how we are going to employ fuzzing software within our software component:
- Awareness of input data structure (model or dictionary based, e.g - dealing with [ASN.1](https://en.wikipedia.org/wiki/ASN.1) parser - fuzzer knows the notation structure)
- Awareness of program nature (program is compiled with especially prepared compiler version which instruments the code)
- Generation or mutation based - differentiates origin of input data - if it is generated on scratch or mutated from some initial 'passing' test case.

For more details please refer to this article on [Microsoft Docs](https://docs.microsoft.com/en-us/previous-versions/software-testing/cc162782(v=msdn.10)). Despite being quite old, it brings terminology and some interesting numbers when comparing and mixing various approaches together basing on simple case study with vulnerable web application prepared on purpose.

## Existing fuzzing approaches versus embedded

Given Microsoft case study above one can clearly see that the best results are when one will combine white box approach (i.e. application or software component is instrumented for purpose of fuzzing) together with smart data awareness (i.e. fuzzer is aware of input data structures).

When it comes to an embedded system we are having three possible approaches:

- Write our application software in portable and compiler independent manner to be able to fuzz it on host platform (however not being able to uncover hardware quirks)
- Emulate given architecture using QEMU (or other emulator) and use black box testing approach - i.e. binary target only without instrumentation
- Write our own software fuzzing framework and use of combined host platform running fuzzing algorithm and trasport protocol to push the data into the embedded device and monitor results

Since the first approach seems to be the simpliest and written software components in portable manner is quite common because of:

- Firmware engineers started to dual targetting reasonable parts of their firmware code even emulating driver layer and RTOS kernels (e.g. FreeRTOS/RT Thread POSIX Ports)
- Model based design generating portable C code is started to being used widely especially within safety critical environments
- In overall it is good practice enabling much more code validation and instrumentation techniques

However using the first approach we must remember that we will leave some gap for the bugs that may had been uncovered when running on real target - but the good thing is that we may took the input which caused the host to fail and then retry it on the same application running real hardware.


## afl - american fuzzy lop

American fuzzy lop is an interesting piece of software developed at Google. It is using genetic algorithms to modify input data and running program is instrumented using especially prepared compiler version (afl-gcc).

1. Simplified algorithm explained (taken out from AFL github readme):
2. Load user-supplied initial test cases into the queue,
3. Take next input file from the queue,
4. Attempt to trim the test case to the smallest size that doesn't alter the measured behavior of the program,
5. Repeatedly mutate the file using a balanced and well-researched variety of traditional fuzzing strategies,
6. If any of the generated mutations resulted in a new state transition recorded by the instrumentation, add mutated output as a new entry in the queue.
7. Go to 2.

## afl - getting started

To start fuzzing working Linux host computer is highly recommended. I did not tested it on MacOS/Windows. My computer is running Ubuntu 20.04.
You will need to resolve some dependencies first:

```
foo@bar:~$ sudo apt update
foo@bar:~$ sudo apt install build-essential && sudo apt install cmake && sudo apt install git
```

It should equip you with everything that you will need to compile basic code in C/C++. You could verify working toolchain by:

```
foo@bar:~$ gcc --version
gcc (Ubuntu 10.3.0-1ubuntu1~20.04) 10.3.0
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

foo@bar:~$ g++ --version
g++ (Ubuntu 10.3.0-1ubuntu1~20.04) 10.3.0
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

foo@bar:~$ make --version
GNU Make 4.2.1
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

foo@bar:~$ cmake --version 
cmake version 3.21.3

CMake suite maintained and supported by Kitware (kitware.com/cmake).

foo@bar:~$ git --version
git version 2.25.1

```

Next step would be to get afl itself, compile and install it. Last step is just to restart the shell.

```
foo@bar:~$ git clone https://github.com/google/AFL.git && cd AFL
foo@bar:~$ make && sudo make install
foo@bar:~$ exec $0
```

After these steps you can verify if everything was properly installed:

```
foo@bar:~$ afl-gcc --version
afl-cc 2.52b by <lcamtuf@google.com>
gcc (Ubuntu 10.3.0-1ubuntu1~20.04) 10.3.0
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

foo@bar:~$ afl-fuzz
afl-fuzz 2.52b by <lcamtuf@google.com>...
...
```

## afl - simple example

Let's start with cloning our vulnerable application repository:

```
git clone https://github.com/Jasinsky/afl_basic_example.git
```

## afl - example with external device

## afl - Moongose networking library - real case

<!-- Interrupt Keep START -->
{% include newsletter.html %}

{% include submit-pr.html %}
<!-- Interrupt Keep END -->

{:.no_toc}

## Further readings

- [Fuzz Testing and Fuzz History](http://secretsofconsulting.blogspot.com/2017/02/fuzz-testing-and-fuzz-history.html)
- [Bart Miller CS Studies 'Project list'](https://pages.cs.wisc.edu/~bart/fuzz/CS736-Projects-f1988.pdf)
- [Microsoft Docs - nice introduction to fuzzing](https://docs.microsoft.com/en-us/previous-versions/software-testing/cc162782(v=msdn.10))
- [Classic afl - american fuzzy lop](https://lcamtuf.coredump.cx/afl/)




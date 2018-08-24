# DarkRISCV
Open source RISC-V implemented from scratch in one night!

## Introduction

Developed in a magic night of 19 Aug, 2018 (between 2am and 8am) the *darkriscv* 
is a very, very experimental implementation of the risc-v instruction set. 

The general concept is based in my other early RISC processors, composed by a 
simplified two stage pipeline where a instruction is fetch from a instruction memory
in the first clock and decoded/executed in the second clock. The pipeline is
overlaped without interlocks, in a way the *darkriscv* can reach the performance of one 
instruction per clock most of time (the exception is after a branch, where
the pipeline is flushed). As adition, the code is very compact, with around one 
hundred lines of verilog code.

Although the code is small and crude when compared with other RISC-V implementations, 
the *darkriscv* has lots of impressive features:

- implements most of the RISC-V RV32I instruction set
- works up to 80MHz and can peaks 1 instruction/clock most of time
- uses only 2 blockRAMs (one for instruction and another one for data)
- uses only around 1000 LUTs

Of course, there are lots of missing features and problems, but they will be 
solved in the future. After the last update, some missing features are now
working fine, such as the load/store w/ 8/16/32-bit support and lots of
fixes regarding the jal/jalr/bcc group of instructions.

Feel free to make suggestions and good hacking! o/

## Implementation Notes

Since my target is the ultra-low-cost Xilinx Spartan-6 family of FPGAs, the project 
is currently based in the Xilinx ISE 14.4 for Linux. However, no explicit references for 
Xilinx elements are done and all logic is inferred directly from Verilog, which means
that the project is easily portable to any other FPGA families.

About the compiler, I am working with the experimental gcc 9.0.0. Although
is possible use the compiler set available in the oficial risc-v site, our
colleagues from lowRISC pointed a more clever way to build the toolchain:

https://www.lowrisc.org/blog/2017/09/building-upstream-risc-v-gccbinutilsnewlib-the-quick-and-dirty-way/

Finally, as long the *darkriscv* is not yet fully working, sometimes is a
very good idea compare the code execution with another stable reference and
I am working with the excelent project *picorv32*:

https://github.com/cliffordwolf/picorv32

One interesting fact is that although the *darkriscv* is 3x more efficient when compared
with *picorv32*, the last one is more heavly pipelined and can reach a clock
2x faster. This means that the *darkriscv* is 1.5x faster than the
*picorv32*, but this result is very preliminary and not really so usefull, as long the 
*darkriscv* is in the very early stage of development. 

As long the motivation around the *darkriscv* is replace some 680x0 and coldfire 
processors, my target is try keep the performance of ~80MIPS in a Spartan-6 LX4.
Unfortunately, after some fixes, I found that a dual-stage pipeline prevents
that the RAM memmory can be handled correctly the load instruction. This
means that the RAM memory must be fully combinational (which means small
memories can work at ~80MHz) or synchronous to the negative edge (which
limits the performance to ~50MHz).

In order to bypass this problem, the pipeline must be updated from 2 stages
to at least 3 states, with an additional stage between the pre-fetch and
execute stages, in order to stop the pipeline when a load/store instruction
is found end enable a variable cycle access (probably 2 cycles when the
access is cached and n cycles when not cached).

## Directory description

- ise: the ISE project files (xise and ucf)
- rtl: the source for the core and soc
- sim: the simulation to test the soc
- src: the source code for the test firmware (hello.c)
- tmp: the ISE working directory (empty)

At the moment, the *darksoc* is not so relevant and the only function is
provide support for the instruction and data memories, as well some related
glue-logic.
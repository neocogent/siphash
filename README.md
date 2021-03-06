# SipHash #
## Introduction ##

This is a hardware implementation of the SipHash [1] keyed hash
function written in Verilog 2001.

The implementation is designed as a self contained core that performs
the message block processing including initialization, compression and
finalization operations. The core does not implement the functionality
to divide a message into 64 bit message blocks.

The implementation supports user defined number of
compression as well as finalization rounds. The core supports all
combinations from SipHash-1-1 to SipHash-15-15.

The core is suitable as an application specific SipHash coprocessor
offloading compact 8, 16 or 32 bit processors from hashing, PRF
generation and Message Authentication Code (MAC) processing. The core is
substantially faster and more compact in terms of hardware resources
than for example cores implementing the MD5 cryptographic hash
function.

The project includes a testbench that verifies that the core generates
the correct response to the testvectors in Appendix A of the SipHash
paper [1]. The project also includes a simple Makefile for compiling the
core using Icarus Verilog [3].

The core has been implemented in an Altera Cyclone IV GX FPGA, see
Implementation notes below for more information.

This core is released as open source under a BSD license, see
LICENSE.txt for more information.


## Usage ##

The core accepts 64 bit blocks (mi) of a given message to process. Prior
to processing a key dependent initalization. Initalization is done by
setting the key port (k) and assering the (initalize) flag for at least
one cycle and then deassert the flag.

There is default number of SipRounds for compression and
finalization. The default values corresponds to the SipHash-2-4
described in the SipHash paper.

Processing a message block is done by assigning the block to the message
block port (mi) and asserting the (compress) flag.

After all blocks in the message has been processed the processing is
completed by asserting the (finalize) flag for one cycle.

The core will assert the (siphash_word_valid) flag when the new SipHash
word for the message is ready.

The core will only accept new commands (initialize, compress, finalize)
when the (ready) flag is asserted.


## Implementation notes ##

The core is implemented using the Verilog 2001 hardware description
language. The core uses synchronous reset for all registers and all
registers are equipped with write enable. The core should integrate and
build cleanly into any standard FPGA project.

The core implements the SipRound function with four 64 bit adders capable
of performing operations in parallel. A single SipRound operation takes
one cycle to perform.

Total latency for processing a message that consists of a single 64 bit
block using SipHash-2-4 is:

 - 1 cycle for initialization
 - 1 + 2 + 1 = 4 cycles for compression
 - 4 + 1 = 5 cycles for finalization
 - 1 + 4 = 5 cycles more for long mode

In total: 10 cycles or 1.25 cycles/Byte.
For long mode 15 cycles or 1.875 cycles/Byte.
For long messages, the latency is asymptotically 0.5 cycles/Byte.


The repo contains both the core itself ([siphash_core.v](https://github.com/secworks/siphash/blob/master/src/rtl/siphash_core.v)) and
a top level wrapper
([siphash.v](https://github.com/secworks/siphash/blob/master/src/rtl/siphash.v)). The
wrapper provides a simple 32-bit interface for the core for easy
integration into a system on chip.


## Implementation results ##

**(Altera Cyclone V)**

- Specific device: 5CGXFC7D6F31C7
- ALMs: 657
- Regs: 867
- No memory blocks, DSPs allocated
- 116 MHz max, slow 85c model


**(Altera Cyclone IV E)**

- Specific device: EP4CE6F17C6
- LEs: 1576
- Regs: 794
- No memory blocks, DSPs allocated
- 101 MHz max, slow 85c model

As a comparison, building the OpenCores MD5 core [2] using the same tools and for the same target device requires the following amount of resources:

- Number of LEs: 1883
- Number of regs: 910
- Max frequency: 62 MHz

Note: MD5 processing takes at least 64 cycles for a message block.


## Status ##

**(2016-05-02)**

Added implementation results for Cyclone IV E. Restored numbers for MD5,
even though the comparison is somewhat irrelevant.


**(2016-04-21)**

The testbench for the top now tests long hash mode, which simulates
correctly. Now we have a core that works for short and long hash modes
and testbenches that perform self checking tests of both modes at core
and top level. Finally a Python model with a good number of tests to
drive verification of the core.

There is also implementation results, see above.

This core is now DONE. Core version has been updated to 2.00.


**(2016-04-20)**

The core now supports 128-bit long hashes as well as 64-bit hashes. The
core generates expected results. There is a first test case for long
hashes in the core testbench.

The Python model supports short and long hashes and uses all test cases
from ([the reference code by Aumasson](https://github.com/veorq/SipHash))

What is left to do:

  - Implement long hash test case in the top level testbench and verify
    that it works
  - Implement all test cases in core and top level testbenches
  - Do test implementations for different FPGAs and update resource and
    performance information
  - Do a real test implementation on a FPGA dev board.

Then the core is really done.


**(2016-04-12)**

The top level now generates tje correct result for the SipHash paper
test vectors. The top level test bench contains self checking test cases
for name and version of the DUT as well as for the SipHash paper test
vector test case.

Next up is fixing the long mode.


**(2016-04-06)**

The core has now actually been debugged and generates the correct result
for the test vectors in the SipHash paper. Amazing that I actually
haven't done this before.

The top level wrapper generates the correct results, but not the way I
expect it too. There is also the beginnings of support for the long
digest mode in the test benches, but core and top lacks functionality
for it.

There has been substantial cleanup work done to the core. It is now much
more compact and readable. I guess one learn by doing stuff...


**(2015-01-24)**

The core now includes the first parts of a beta implementation of the
long version with 128 bit digest.



## Contact information ##

For any questions and inquiries regarding the siphash core, please
contact Secworks Sweden AB: http://secworks.se/


## References ##

[1] J-P. Aumasson, D. J. Bernstein. SipHash: a fast short-input PRF.

  - SipHash Project: https://131002.net/siphash/
  - Siphash Paper: https://131002.net/siphash/siphash.pdf


[2] OpenCores. MD5 core.

  - Core home page: http://opencores.org/project,systemcmd5


[3] Icarus Verilog

  - http://iverilog.icarus.com/

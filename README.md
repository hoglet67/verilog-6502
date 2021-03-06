========================================================
A Verilog HDL version of the old MOS 6502 and 65C02 CPUs
========================================================

Original 6502 core by Arlet Ottens

65C02 extensions by David Banks and Ed Spittles

==========
6502 Core
==========

Arlet's original 6502 core (cpu.v) is unchanged.

Note: the 6502/65C02 cores assumes a synchronous memory. This means
that valid data (DI) is expected on the cycle *after* valid
address. This allows direct connection to (Xilinx) block RAMs. When
using asynchronous memory, I suggest registering the address/control
lines for glitchless output signals.

[Also check out my new 65C02 project](https://github.com/Arlet/verilog-65c02)

Have fun. 

==========
65C02 Core
==========

A second core (cpu_65c02.v) has been added, based on Arlet's 6502
core, with additional 65C02 instructions and addressing modes:
- PHX, PHY, PLX, PLY
- BRA
- INC A, DEC A
- (zp) addressing mode
- STZ
- BIT zpx, absx, imm
- TSB/TRB
- JMP (,X)
- NOPs (optional)
- 65C02 BCD N/Z flags (optional, disabled)

The Rockwell/WDC specific instructions (RMB/SMB/BBR/BBS/WAI/STP) are
not currently implemented

The 65C02 core passes the Dormann 6502 test suite, and also passes the
Dormann 65C02 test suite if the optional support for NOPs and 65C02
BCD flags is enabled.

It has been tested as a BBC Micro "Matchbox" 65C02 Co Processor, in a
XC6SLX9-2 FPGA, running at 80MHz using 64KB of internel block RAM. It
just meets timing at 80MHz in this environment. It successfully runs
BBC Basic IV and Tube Elite.

============
Known Issues
============

The Matchbox Co Processor needed one wait state (via RDY) to be added
to each ROM access (only needed early in the boot process, as
eventually everything runs from RAM). The DIHOLD logic did not work
correctly with a single wait state, and so has been commented out.

I now believe the correct fix is actually just:

always @(posedge clk )
    if( RDY )
        DIHOLD <= DI;
 
assign DIMUX = ~RDY ? DIHOLD : DI;

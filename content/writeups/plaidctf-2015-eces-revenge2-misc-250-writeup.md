---
title: "PlaidCTF 2015 - ECE's Revenge2 (misc 250) Writeup"
slug: "plaidctf-2015-eces-revenge2-misc-250-writeup"
date: "2015-04-22 23:30:46.569557"
author_name: "FalconK"
author_email: "falcon@iridiumlinux.org"
draft: false
toc: false
images:
---

by FalconK

This is an extremely simple problem, once you get past the need to find
a System Verilog program.

- Part A is a D flip-flop.  The latch disable bit is unused here.

- Part B is a half-adder.

- Part C is a multiplexer.  The strobe is not used here.

- Part D is a counter.  The initialization function is not used here.

If you look through the verilog definitions, you will notice that the
important inputs are latched.  Win latches to high if it becomes high at
all, so you don't actually have to time it to happen at the particular
clock cycle the hint in the comment specifies.  Having it latch high at
any time is fine.  It turns out that it will latch high after the 31st
(last) bit of the input stream is consumed, if it was right.

The other bit that latches in once high is the w_t bit.  This is the
fail bit; if any of the input bits are wrong, it will latch high and
prevent win from becoming set.  Win is set whenever the multiplexers are
pointing to the last input bit (the selector is 0b00011111), and w_t is low.

A short inspection will show that the counters are simply monotonic on
the clock, and read the input bits sequentially.  The current input bit
is m_f.

Recall that our goal here is basically to keep w_t low until the counter
gets to 31.  Reading the verilog shows is that w_t becomes high when
im_f XOR m_o - that is, whenever the input bit and m_o are not the same.

That's actually as far as we need to go.  Using the simulated logic
analyzer, watch the value of m_o.  Set im_f, from the low bit to the
high bit, to the value of m_o as soon as they differ (and notice that
when they do, w_t becomes and stays high, and win remains low).  m_o
pretty much reads the key.

When you have the right sequence, which happens to be
0b11011000010100110010000100010000, you have the flag (it's just
"11011000010100110010000100010000").  You'll notice that when you have
that set, the w_t bit stays low, and on the cycle that reads the 31st
bit, win goes and stays high forever.

This problem is sufficiently simple that with no prior knowledge of
verilog, I solved it with ModelSim (in about 6 hours but oh well).  You
have to remove the first line of the verilog file, which is a syntax
error in ModelSim; I assume that was an artifact and not actually part
of the problem.

Here's an example of how I did it.  Open the verilog file and remove the
first line (which has a syntax error and is not needed).  Compile it,
and make sure the module you are simulating is "testbench".

Add the outputs you want to the waveform by right-clicking on them in
the list of outputs.  Make sure the simulation runtime is high enough (I
am using 500ns).  Then, run it, and observe what happens when im_f and
m_o differ.

Count out and flip that bit of user_in, recompile, reset the simulator,
and run.  Many things change (including the m_o waveform).

You will have to go through several iterations of this, but in that
screenshot, even though only one bit changed, the entire string was
correct (notwithstanding that im_f differed from m_o in several bits in
the last iteration).

The reason this works is that until it is called for in the cycle, each
bit has no effect on the circuit because the 32-input multiplexer (made
of five 8-input ones) has only one output, and its selector increments
monotonically.  Since a single wrong bit immediately latches us into the
fail state forever, the solution must be to flip bits to avoid that
state, one cycle at a time, until the only possible solution is found.

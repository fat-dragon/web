---
title: "Programming the Unix TTY Abstraction"
date: 2019-01-04T15:43:55Z
---

Let's talk about how Unix handles input and output, and do some
programming.

As I discussed in the post on [terminals](/post/terminals/), early
Unix system used repurposed electro-mechanical hardcopy
teletypewriters as terminals.  The user sat in front of one of
these, typed on it, and the characters s/he entered were transmitted
to the host computer (running Unix, of course; the computer in this
case being either a PDP-7 or a PDP-11).  Output from the host was
transmitted to the terminal, which printed it onto a (long) scroll
of paper.

But what we didn't talk about was _how_ Unix makes all of this
happen.  Before we go further, let's get a handle on that first, so
let's talk about I/O.

## Polling, Interrupts, DMA, and Peripherals

A computer system consists of several parts.  The "brain" of the
machine is the central processing unit: this is the unit that
actually performs computation; it has an arithmetical logic unit
(ALU) for doing arithmetic and some amount of supporting circuitry.
Generally what it does is take instructions from memory (more on
that in a moment) and act on them.  Instructions, in turn, are small
bits of data that represent commands to the CPU: "Add 1 to the
number stored in register 0"; "test where the contents of register 0
are greater than 10 and set a status bit"; "move the contents of
register 2 to local 10011100 00110100 in main memory"; etc.
Instructions are encoded into a small number of bytes or words and
these are what the CPU actually deals in: the CPU fetches them,
decodes them, executes them, and stores the results.

As programmers, we tend to write code in high-level languages (such
as SML, or C, Go, Rust, Pascal, Python, Lisp, JavaScript, etc) and
use either interpreters or compilers to take the symbolic, textual
representation of our programs and distill them down to instructions
that can actually be executed by the computer.

Implicit in this is that the computer must also have some amount of
memory to hold interesting data and instructions.  We often refer to
this as "main" memory.  Circuitry is provided to connect the CPU to
this memory store.  Main memory can be directly addressed by the CPU
and falls into two categories: either "random access memory", or
RAM, which tends to be readable _and_ writable, or "read-only
memory" or ROM, which as it's name implies is readable but not
writable.  Special instructions exist for copying data between the
CPU and main memory; note that the CPU itself often has a small
number of word-sized memory locations called _registers_ that it can
use to hold data it is working on.  A typical computational sequence
might involve an instruction to move data from main memory into a
register, a sequence of instructions to manipulate that data in some
manner, and another instruction to copy the result from the register
back into main memory, possibly overwriting the original data.  Some
computers feature instructions that can manipulate data directly in
memory, while others require copying it into a register first.  Note
that memory exists in some address space, and most modern machines
provide two address spaces: a _physical_ address space that deals
with memory as it is placed in the computer by the hardware
engineers, and a _virtual_ address space, which puts a layer of
indirection between a program's view of memory and the physical
memory in the machine.  Note that this is what "virtual memory"
means; it does not necessarily mean handling overruns of the
physical memory store by paging or swapping bits of a program's
executing image to disk.  A dedicated piece of hardware call a
"memory management unit" or MMU is responsible for translating from
the virtual to the physical address spaces.

But by themselves, a CPU and memory aren't particularly interesting;
sure, you can compute with them...but what do you do with the
results?  How do you get access to interesting data to perform
computations on?

Enter the concept of input and output; we want to connect our CPU
and main memory store to some other device which will provide us
data to work on and allow us to either present the result to a user
or store it somewhere.  These are referred to as _peripheral_
devices because they exist on the periphery of the computer, outside
of the CPU and main memory store.  The act of retrieving data from a
peripheral is input; the act of transmitting data to a peripheral is
output.  Input/Output or I/O or IO are important classes of
operations in a computer.

Now the question becomes, how do we perform IO?

Some machines provide special instructions for interacting with
peripherals; the IBM PC falls into this category.  This is called
"programmed IO".  Others make a control interface for a device
appear in the same _address space_ as the main store; one interacts
with the device by treating it like memory and using memory transfer
instructions to program the device itself; this is called
"memory-mapped I/O".  The trend in modern systems is to toward
memory mapped IO, but systems that provide programmed IO
instructions often use a mix of the two approaches.

At first, memory mapped IO can be a little hard to understand: after
all, if the peripheral looks like memory, how is it any different
from actual memory?  The critical thing to understand is that the
thing that looks like memory from the perspective of the host
computer is often only the _control_ interface for the peripheral.
By way of example, consider an SSD or other "mass-storage" device.
The total amount of data one can store on such a device may exceed
the total expressible size of the host machine's memory address
space (e.g., a 32-bit processor can address 4GiB of memory, but
that's a pretty small SSD or disk or even SD card at this point).
The disk only exposes a small control interface to the address space
of the machine; through carefully sequenced memory reads and writes
to that interface, one can cause the SSD to retrieve a small block
of data that it then copies into the main memory of the host.  We
_control_ the device by the memory mapped IO region, but the device
is free to implement storage or whatever it likes any way it
chooses.

Which leads us to another consideration: _how_ do we transfer data
to and from the device?  Again, we have two choices: we can either
do this explicitly, using messages we exchange with the device, or
we can tell it where the data we want to transfer is or should go.
For simple character-at-a-time data, such as reading from a serial
port, the former is perfectly acceptable: but when we start taking
about large amounts of data like we would get from an SSD or network
interface, the overhead becomes overwhelming and we usually take the
latter approach.  We can set up a transaction to retrieve a block of
data from the SSD and write it into a set location in the main
store; this is called "direct memory access" because the peripheral
directly accesses the store without the CPUs involvement.  This is
the preferred mechanism for transfering large amounts of data
between peripherals and main memory and/or registers.

But wait, this raises another question: how do we know when the data
has been transferred?  For that matter, for a device like a serial
port, how do we know that data is available to be transferred, or
that the UART is done transmitting a character so we can start
transmitting the next character?

Again, there are two methods: the control interface for a device
usually provides us some sort of status indicator for detecting
these things: we can simply have a program sit in a loop testing
that indicator.  Assuming the CPU is faster than the peripheral,
we'll known as soon as the peripheral is done and be able to do
whatever comes next: either retrieve data, initial the next
transfer; whatever is required for the task at hand.

But this is wasteful.  If we the CPU is much faster than the
peripheral device, we'll spend an inordinate amount of time polling
IO status, taking time away from doing other interesting work.
Sure, we can test only every so often, like when the program hits
some convenient point, but then we may miss events because now we
may not be serving the peripheral fast enough (we might miss a
character coming in from the serial port, for example).

The solution is to use a mechanism known as an interrupt: this is
like the peripheral giving the CPU a tap on the shoulder when it has
done something.  We can program the peripheral to deliver an
interrupt to the CPU when it's completed a DMA transfer to or from
an SSD, or when it receives a character at the serial port.  On
receipt, the CPU can pause what it is doing, acknowledge the
interrupt, serve the IO action, and then resume whatever it was
doing.  This allows the CPU to spend most of its time on computation
while still providing us a mechanism to handle IO.  The downside is
greater programming complexity and higher _latency_: it takes time
to set up the peripheral to deliver an interrupt, time for the CPU
to save and restore its _context_ when handling an interrupt, and
time to _serve_ the interrupt.  Whereas if we poll we know instantly
when an operation has completed, when using interrupt we have to add
all these times into the mix.  On the balance, however, that's not
so bad.

This is all a bit hand-wavey and of course modern systems are more
complex than this simple sketch of a computer.  They may contain
more than one CPU, and they may decide to sacrifice one or more CPU
cores to polling in order to cut down on latency, they may contain
multiple levels of cache over the main memory store to boost
performance, etc.  Suffice it to say that the tradeoffs can be
complex, but interrupts and DMA give hardware and software system
designers great flexibility.  For a rough-cut presentation, that's
the gist of how systems actually work.

But I, as a programmer do not, and should not have to, care. Sure,
knowing a little bit about the underlying system organization is
useful, but it's not something I want to write code against.

Instead systems software, like operating systems (such as Unix)
provide abstractions that I, the programmer, write software against;
it is the responsibility of the operating system to translate those
abstractions into actions on physical hardware.  The implementation
of this will probably make use of a _device driver_, or bit of code
responsible for running the actual peripheral.  The implementation
of the abstract interface I program against will work with the
driver to communication with the peripheral and my code will be
decoupled from the vagaries of the underlying device and all the
associated complexity.  The operating system will be free to
interleave computation with IO in whatever way makes sense to
maximize resource utilization on the system, etc.

## Back to the Unix TTY Abstraction

So what does any of this have to do with Unix?

On Unix, to a first order approximation, all devices are made to
look like files.  Recall that in the early days of Unix, all access
to the machine was via serial terminals.  A serial port connected to
a terminal is associated with a special "device file" in the
filesystem and if I want to read data from that serial port, I open
that device file and issue a _read_ system call against it: a
system call is a way for a user-level program to request service
from the operating system.

The operating system will see that I'm reading from the serial port
and, if data is available for me to consume, it will return it to my
program.  But Unix has been multiprogrammed almost from the
beginning, so if data is not available, it will _block_ my process
until some becomes available and then invoke the _scheduler_ to go
find something else useful to do with the CPU while I wait.  In this
way, asynchronous IO from the serial port is made to appear
synchronous as far as my program is concerned, but the system isn't
blocked by my program sitting around waiting for the user to decide
to do something.  The details of waiting for data to consume and
multiprogramming the CPU are all handled by the operating system and
are independent of my program: I, the programmer, don't have to
care.

The software in Unix that does this is the TTY subsystem, so named
because of the use of teletypewriters as early terminals for Unix
systems.  The TTY subsystem provides the programming interface I
write software against and interfaces with the device drivers that
handle the actual hardware.  When a byte is received at a serial
port, hardware may generate an interrupt that causes the serial
device driver to run.  The driver will retrieve the byte and provide
it to the TTY subsystem that stores it in a buffer inside the
operating system.  When my software goes to read that byte, it is
removed from that buffer and copied into my process.  When I write a
byte from my program to the terminal, it is copied into another
buffer in the TTY subsystem and when the serial port is ready to
transmit that byte, it might again generate an interrupt to let the
host know that it's ready; the driver can then get the byte to be
transmitted from the TTY subsystem and start transferring it.  The
TTY abstraction insults me, the programmer, from the details of
interrupt handling and buffer management.

But there's more.  Since early Unix systems were accessed by serial
terminals, most serial access to the machine was interactive and it
turns out that there are a lot of operations that are common across
interactive programs.  If I write a simple program that wants to
read a string from the user and echo that back, then I shouldn't
have to care if the user makes a typo, hits the backspace key a
couple of times, and corrects the typo.  It's not relevant to the
program _I_ am writing, so I'd like my abstraction to handle those
sorts of details for me.  Furthermore, I don't necessarily want my
_read_ call to return as soon as a byte of data is available;
instead, I may want to wait until an entire line of data is ready to
be consumed by my program, or at least until some reasonable sized
buffer fills up.  Again, these are the sorts of messy details I want
my programming abstraction to handle for me: spare me from the
minutiae.  The TTY abstraction does this, and the result is a
default _line discipline_ that interposes between my program and the
underlying hardware device.  If I type a backspace key to back over
a typo, by default that is handled by the TTY subsystem, which
removes the last item from the input buffer, before it gets to the
user program.  When I hit the "Enter" key, if my program is waiting
for a line of data, it will be copied into my process and my _read_
call will complete.  Similarly, if I type Ctrl+C to interrupt (not
to be confused with hardware interrupts!) the currently running
program, the TTY subsystem will receive the character and, instead
of copying it into a buffer, it will cause a SIGINT signal to be
delivered to my program instead.  Some terminals also require a
two-character carriage-return/line-feed sequence to advance to the
left margin of the next line, but Unix programs usually just emit a
linefeed character.  The line discipline would handle things like
inserting carriage-returns, handling tab stops, etc.

Using the default line discipline is usually referred to as being in
_canonical_ or _cooked mode_, but there are _raw_ modes for when I
don't want line discipline handling (such as in a display editor
like _emacs_ or _vi_, or in libraries that handle the terminal
specially like _curses_ or _readline_).

When Unix graduated into the world of networking, the TTY subsystem
was extended so that network connections could be connected to
"pseudo-terminals", which provided the TTY abstraction but talked to
a program that communicated with the network on the other end
instead of a serial driver.  A program run by a user connecting via
an SSH connection may behave as if is communicating with a TTY, but
the TTY device in question is synthesized by the operating system to
allow the software to make use of the same programming abstractions
used by programs designed to work with serial terminals.  Thus
programs can continue to make use of the default terminal line
discipline, etc, while being served by some network protocol like
SSH or TELNET.  Similarly with programs running in terminal programs
in windows on bitmapped graphical displays.

The TTY subsystem can be programmed to precisely control the
terminal's behavior, though the programming interface has changed
significantly over time.  In the early days, one's TTY could pretty
much be in canonical mode or raw mode, and one could set a few
parameters such line speed, what character your terminal sent when
you hit the _erase_ key, etc, but that was about it.  Berkeley
variants introduced a mode somewhat in between canonical and raw
mode called _cbreak_ mode, in which character-at-a-time input worked
without the TTY accumulating an entire line of text, but processing
of control characters like Ctrl+C continued to work as expected and
generate signals.

Ever more elaborate interfaces were proposed for different variants
of Unix until the POSIX standard introduced the
[termios](http://pubs.opengroup.org/onlinepubs/009696699/basedefs/termios.h.html)
standard, which is ubiquitous now.  This allows us to write portable
programs against the TTY abstraction.

Of course, this isn't a panacea: the TTY abstraction tends to leak
all over the Unix (and Linux) kernel and even in the 1980s was
something of an anachronism as fewer and fewer users were using
serial terminals anymore.  Job control (the ability to suspend
processes and run them in the background and move things between
foreground and background) and POSIX "sessions" made it even more
complex.  But none of that is particularly germane to what we're
doing here and is
[explained](https://www.linusakesson.net/programming/tty/) elsewhere
for the curious reader.  For now, let's take the interface as-is
and move on to programming it.

## Programming the `termios` Interface from SML

The SML basis library provides us an interface to the
[POSIX TTY](http://sml-family.org/Basis/posix-tty.html)
abstraction.  For our BBS application, we want to use this to put
the terminal into some mode so that we can read characters
immediately as they are typed by the user, instead of waiting for
them to be accumulated into a line of text.  However, we don't want
complete raw mode because we'd like to do things like strip carriage
returns from input and automatically insert them on output instead
of doing that ourselves.  The infrastructure is there: the basis
library support presents a number of structures to control the input
and output processing functions of the terminal, as well as device
and line discipline controls.

Finally, we want to preserve the old terminal settings so that we
can restore them when our program exits: changes to the TTY
configuration persist across program invocations, since the TTY
is associated with our login session, not just one running process.

However, the basis library is just a thing wrapper around the POSIX
C interface and is a bit cumbersome.  We'll put some scaffolding
around this to make it a little easier to use.

The first thing we do is define a structure to encapsulate our
terminal related code, and put in a function that will take a file
descriptor referring to a TTY device that configures the device in
the desired way and returns the old mode.  We'll similarly provide
a function to reset the mode on the TTY device to some saved value.

```sml
structure Terminal = struct
    fun setTermMode fd =
        let
            open Posix.TTY

            (* Retrieve existing terminal attributes *)
            val attr = TC.getattr fd

            (* Set up raw mode attributes here *)
            val rawModeAttr = ...
        in
            TC.setattr (fd, TC.sanow, rawModeAttr);
            attr
        end

        (* Resets terminal attributes to specified values. *)
        fun resetTerm fd attrs =
            Posix.TTY.TC.setattr (fd, Posix.TTY.TC.sanow, attrs)
end
```

Here what we're doing is retrieving the current terminal attributes,
creating a "raw" mode attribute, changing the current terminal
settings to those raw mode attributes, and then returning the
original (unmodified) attributes.

What does the code for setting up the raw mode attributes look like?
Actually, fairly simple, if somewhat tedious.

As mentioned, the `termios` interface provides us with four main
categories of attributes that we can manipulate: separate attributes
for input and output processing, for controlling device settings
(line rate, data size, parity etc) and line discipline settings.
These are bundled up into an SML _record_, which is what is returned
by the `TC.getattr` function call.  These sets of attributes are
represented as bit flags using the
[BIT_FLAGS](http://sml-family.org/Basis/bit-flags.html) signature.

What our code is is retrieve each of these in turn, then, for each
category of attribute, we create a temporary datum representing the
_compliment_ of attributes we wish to disable for that category.
We clear those, enable any relevant attributes, and store the result
in a temporary.  Finally, we assemble these temporary attributes
into an attributes record, along with things like input and output
speeds (that we simply carry over from the existing attributes)
and that's what we pass to `TC.setattr`.  Here is the full code
for `setTermMode`:

```sml
fun setTermMode fd =
    let
        open Posix.TTY

        (* Retrieve existing terminal attributes *)
        val attr = TC.getattr fd

        (* Functionally create terminal attributes for raw mode *)
        val attrIFlag = Posix.TTY.getiflag attr
        val rawIFlagC =
            I.flags
                [ (*I.maxbel,*) I.ignbrk, I.brkint, I.parmrk,
                  I.istrip, I.inlcr, (*I.igncr, I.icrnl,*) I.ixon ]
        val iflag = I.clear (rawIFlagC, attrIFlag)
        val iflag = I.flags [ iflag, I.icrnl, I.igncr ]

        val attrOFlag = getoflag attr
        val rawOFlagC = O.flags [ (* O.opost *) ]
        val oflag = O.clear (rawOFlagC, attrOFlag)

        val attrCFlag = getcflag attr
        val rawCFlagC = C.flags [ C.csize, C.parenb ]
        val cflag = C.clear (rawCFlagC, attrCFlag)
        val cflag = C.flags [ cflag, C.cs8 ]

        val attrLFlag = getlflag attr
        val rawLFlagC = L.flags [ L.echo, L.echonl, L.icanon, L.isig, L.iexten ]
        val lflag = L.clear (rawLFlagC, attrLFlag)

        val attrCC = getcc attr
        val rawCC = [(V.min, chr(1)), (V.time, chr(0))]
        val cc = V.update (attrCC, rawCC)

        val rawModeAttr =
            termios {
                iflag = iflag,
                oflag = oflag,
                cflag = cflag,
                lflag = lflag,
                cc = cc,
                ispeed = CF.getispeed attr,
                ospeed = CF.getospeed attr
            }
    in
        TC.setattr (fd, TC.sanow, rawModeAttr);
        attr
    end
```

And that's basically it.  How did we know what attributes to clear
and what to set?  In part by careful reading of the POSIX
specification.  The standard C library also includes a function
called `cfmakeraw` that we can look at to see what it sets and
clears.  Our mode isn't exactly raw mode as produced by
`cfmakeraw`, but rather a modification that leaves some processing
enabled such as carriage-return suppression on input and insertion
on output.  `maxbel`, which suppresses beeping the terminal bell
when the input buffer is full, isn't defined in SML so we ignore it
even though the C library clears it.

Anyway, now we can make use of this in a program:

```sml
fun main() =
    let
        val savedTermMode = Terminal.setTermMode Posix.FileSys.stdin
        val vec = Posix.IO.readVec(Posix.FileSys.stdin, 16)
    in
        print ("Read: |" ^ (Byte.bytesToString vec) ^"|\n");
        Terminal.resetTerm Posix.FileSys.stdin savedTermMode
    end

val _ = main()
```

Note that we read a _vector_ of bytes here since the user may type
a key that yields a multibyte input sequence (e.g., an arrow key).

Compiling and running this on the Fat Dragon gives the expected
result:

```text
: fat-dragon; mlton terminal.sml
/usr/local/lib/libgmp.so.10.0: warning: vsprintf() is often misused, please use vsnprintf()
: fat-dragon; ./terminal
Read: |a|
: fat-dragon; ./terminal | viz
Read: |^[[A|\n
: fat-dragon;
```

Note that ": fat-dragon; " is my prompt.  In these two runs, I typed
the letter 'a' and an up-arrow, respectively.

Now we have some basic terminal handling.  Next up is building a
mechanism for text-file based configuration.

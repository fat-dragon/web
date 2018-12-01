---
title: "Unix"
date: 2018-12-01T02:27:28Z
---

Our system of choice is based on Unix.  So let's talk about Unix.

## History

In the beginning was a batch mode system, the "Fortran Monitor
System" for the IBM 709 computer at the MIT computation center.
Experiments in timesharing where multiple users interactively shared
the machine's resources were done on the 709, which was was then
upgraded to a 7090.  IBM Flexowriter typewriters were connected to
the 7090 for use as (hardcopy) terminals, and this begat the
Compatible Timesharing System CTSS ("compatible" because it
co-existed with the FORTRAN monitor), which spent most of its life
under the auspices of MIT's project MAC.

This generated much excitement; enough that MIT entered into a
partnership with then computer manufacturer GE and Bell Labs to work
on a new timesharing operating system that they envisioned as a
"computing service": a single machine with enough computing power
that it could serve all users in the city of Boston.  Visions of
users plugging terminals into sockets in one's wall to gain access
to computational resources, much in the way that one did with a
telephone or electrical appliance.  A true computer utility.  This
was MULTICS, the "Multiplexed Information and Computing Service";
note the name ends in "-ICS", not "-IX".  Work started was in the
early/mid 1960s targeting the GE-645 computer, which was a GE-635
enhanced with hardware support for virtual memory and paging.  The
645 was a 36-bit, word-addressed machine.  The 36 bit word size was
not unusual at the time (recall that early digital computers were
competing against mechanical calculators, which had 9 digits of
precision; 36 bits allows for 9 BCD digits per word).

Two Bell programmers who worked on Multics were fellows by the name
of Dennis Ritchie and Ken Thompson, but individuals such as Doug
McIllroy, Brian Kernighan, Joe Ossanna, Rudd Canaday and others were
among the cast of characters dialing (initially) into the CTSS
system from Bell Labs in Murray Hill, NJ and then calling into
Multics itself.

By the late 1960s, it was clear that Multics would not deliver on
its grand vision in a timely manner and Bell Labs made the decision
to leave the project.  This left the BTL researchers without a
comfortable computing environment: Multics was slow and expensive at
the time, but it was much nicer to use than contemporary systems of
the day.

One of those programmers, Ken Thompson, had written a game he called
"Space Travel": it was a "faithful" simulation of the solar system
in which the player piloted a small spaceship landing on different
planets and moons, etc. It ran poorly under Multics; response was
slow and jerky, and runs were expensive and a single run might cost
$75: of course, this was unrealized mainframe funny-money
accounting, so no one actually _paid_ $75, but still, it was not
cheap in terms of resource utilization on the shared machine.

One day, Thompson found a cast-off DEC PDP-7 minicomputer in the
corridor of the BTL facility; it was 1969.  The PDP-7 was antiquated
even for the time, but it had an excellent vector graphics display
facility.  Thompson ported Space Travel to this machine, then had
the idea to build a small operating system for it, based on the
design for a filesystem he had sketched out the previous year with
Canaday and Ritchie.  Thompson's wife, Bonnie, went on vacation for
a month that year and Thompson gave himself a week each to write the
shell, editor, kernel, and assembler for the a self-hosting system
on the PDP-7.  He was successful and quickly attracted the attention
of Ritchie and others in their group, mostly former Multicians.

The result was a small operating system that Brian Kernighan
jokingly referred to as "UNICS", a pun on "Multics" that was
allegedly a portmanteau of "Uniplexed Information and Computing
Service", but since it vaguely sounded like "eunuchs" also somewhat
basely referred to as, "a castrated version of Multics": the times
were not known for their enlightened sense of humor.

Much interesting work was done on this small machine, including the
invention of a language called "B", which was a cut-down version of
BCPL used as a systems language.  However, the PDP-7 was too
constrained of an environment to do much interesting computer
science on, but enough had been done that the nascent group could
successfully lobby Bell Labs management to approve the installation
of a DEC PDP-11/20 minicomputer for operating system research and
development.  By this time marketing had caught wind of the work
and, while the Bell system was legally prohibited from entering the
computer industry due to a 1955 consent decree granting it monopoly
status over the telephone network, they nonetheless decided that the
name was inappropriate, either because of the eunuchs jokes or
because of the overt similarity to Multics (which was very much
still a going concern outside of BTL).  They insisted that the name
be changed to "Unix", which stuck.

Unix of course eventually took on a life of its own and progressed
through several "research" editions, eventually being rewritten in
the C programming language, which evolved from B by adding types to
the earlier language.  It eventually escaped the lab, was ported to
a myriad different hardware platforms, and subsequently gave
inspiration to a young Finnish student to write a clone that he
named after himself: "Linux".

But that's another story.

## What makes Unix Useful?

Unix is useful for us for a few reasons.  Recall that in the article
about our goals, we discussed how we were interesting in exploring
how systems that were more sophisticated than DOS, CP/M, etc, could
be used to enhance the BBS experience.  We want a system that can be
used for _more_ than just the usual BBS functions of menu-driven
messaging, games, and file transfer.  Moreover, we want to leverage
more of the underlying system.

One of the most useful features is that Unix is inherently
multiprocessing: that is, the system has a notion of processes,
which can be thought of as a program in some state of execution.
Multiple processes can be in a state of "running" at a time, giving
us concurrency.  That is, the system is multiplexed between multiple
programs, all of which appear to be running simultaneously.  This
means that it can support multiple "lines" hosting multiple users at
one time, which no special programming required.  Furthermore,
processes can communicate using well-defined interfaces like pipes
and sockets, and they're isolated from one another: a fault in one
process doesn't generally affect another.

Furthermore, the system is inherently multiuser: support for
multiple users accessing the system simultaneously has existed since
the PDP-7 days.  Moreover, different user accounts provide security
barriers between one another: users can't arbitrary influence
another user's processes, or access someone else's files.  In this
sense, accounts can be employed as protection domains.  A dedicated
user can host a message conference, owning all of the associated
files, etc, but provide access to other users via an IPC mechanism
based on sockets or (named) pipes.  Importantly, the base operating
system can be protected from ordinary users; having access to the
shell doesn't confer unrestricted access to the computer.

As one might expect from a mature operating system that is nearly 50
years old, most Unix distributions come with a healthy complement of
software in the form of utility and administration programs,
editors, shells, software development tools, etc.  With the rise of
Linux, many more are available and easily installable.

The system is relatively conceptually simple: to a first order
approximation, everything in Unix is represented or accessed like a
file.

Of course, like anything, the system is not perfect.  Many have
argued, correctly, that neither graphics nor networking were
gracefully integrated into Unix: the sockets interface is the
predominate networking API and is inelegant and not very "unixy" in
that sockets exist in their own namespace outside of the filesystem.
Yes, one can transfer data on a socket using the normal _read_ and
_write_ system calls, but control operations on sockets involve
dedicated system calls.  X11, the most commonly used window system,
was a research prototype and again doesn't fit onto the underlying
system very elegantly: it exists entirely outside of the filesystem.

As an interesting historical side note, as the BBS scene was really
picking up steam in the mid- to late-1980s, the 1127 research group
at AT&T Bell Labs, which originally developed Unix and C, felt that
they had taken Unix about as far as they could as a vehicle for
computer science research.  They moved on to a new system called
Plan 9 that arguably fixed many of Unix's problems: resources on
plan9 are represented as filesystems and operations involve
manipulating a per-process (group) filesystem namespace to hold the
collection of filesystems useful to the user for a particular
operation, and then acting on those resources using a small set of
primitives.  There is a single file service protocol called "9P"
that is used ubiquitously to serve filesystems to processes.  In
these filesystems, "files" and "directories" might not represent
anything backed up by a stable storage device such as a disk, and
may be entirely synthesized by the operating system or by a user
program that generates 9P.  For example, networking is a filesystem,
and opening (say) a TCP connection to a remote host involves opening
files and reading or writing to them.  Similarly, the window system
is a filesystem that itself wraps over files representing the
keyboard, mouse and graphics device; this made it easy to distribute
over a network. Interestingly for each window, the window system
re-exposed (synthesized) windows representing the keyboard, mouse,
and a graphical device representing the window itself.  This meant
that the window system could be run recursively.  Sadly, Plan 9 was
not a sufficiently great leap forward from Unix to supplant its
ancestor, but it was a very interesting system.
I [wrote](http://pub.gajendra.net/2016/05/plan9part1) more about it,
and a [mirror](https://9p.io/plan9/) of the Bell Labs web site is
available.

Getting back to Unix, perhaps one of the ugliest but most useful
(for our purposes) aspect of Unix's design is the terminal
subsystem.

## Terminals

Unix dates from a time when hardcopy terminals were still the most
common way to access a timesharing computer.  Thus, the "TTY" (or
teletype) is a fundamental abstraction in most Unix kernels: it is
_the_ interface by which the user interacts with the machine.  In a
sense, every Unix user is accessing the system "remotely" via a
terminal.

In the early days, each TTY device represented a serial port on the
host computer and was connected to a terminal or a modem.  One could
connect as many terminals or modems to a Unix host as one had serial
ports for.  Anything beyond that would require additional hardware,
but in those days, multiple ports was extremely common.

To manage this, a terminal driver was written.  This interacted with
the driver for the underlying serial device and imposed behavior on
the line to do things like ensure that characters received from the
serial port were echoed back to the terminal, translate line feed
characters sent from the host to sequences of carriage returns and
line feeds (recall that hardcopy terminals had a print head that
would have to be returned to a margin after emitting a line of text)
etc.  Most terminal drivers provided facilities for accumulating a
logical line of text, including handling of erase characters, tab
stops, etc, before returning input to a user process; this cut down
on system call overhead and allowed for some amount of editing in
the terminal itself, before presentation to a program.

A "raw mode" was included that allowed the terminal system to take
over a serial line and use it for pure communication; this
suppressed any of the handling just described.

Eventually, hardcopy terminals were replaced by cursor-addressed,
"green screen" terminals but approximately the same model was
preserved.  For applications taking advantage of the richer
presentation model presented by these terminals "raw" mode was
import to allow the program to receive individual characters one at
a time, etc.

## The Rise of Networking and Graphics

In 1975, Ken Thompson took a sabbatical year at the University of
California, Berkeley.  Berkeley had a PDP-11.  Thompson brought a
tape containing Unix.

Unix went onto the Berkeley PDP-11 and Thompson started writing a
Pascal environment for it for teaching.  Chuck Haley and Bill Joy,
two graduate students, soon joined the Pascal effort.  Eventually,
Berkeley started working on Unix and producing the "Berkeley
Software Distribution" of Unix, or BSD.  A globally important
development was the addition of TCP/IP and the "sockets" API in
versions of the 4th BSD distribution, or 4BSD.  In the early 1980s,
4BSD was selected by administrators at the Defense Advanced Research
Projects Agency of the US government as a standard platform for
contractors connected to another research network called the
Internet, with the hopes that having a common computing platform
would encourage research collaboration and reuse.  It did, and
TCP/IP and the Internet became wildly popular, eventually forming
the backbone of our modern interconnected information
infrastructure.  As a consequence, Unix systems have supported
TCP/IP networking natively for 30 years.

Similarly, workstations with bitmapped graphics displays were
appearing on the scene, freeing users from the shackles of
centralized computers and "green screen" terminals with limited
functionality.

## Where is the Terminal Abstraction Now?

With the rise of networking and bitmapped graphics displays this
facility made less sense.  Regardless, starting with networking, the
basic infrastructure was recycled by the introduction of a
"pseudo-terminal" device.  Instead of being connected to a physical
piece of hardware, a pseudo-terminal is synthesized by the operating
system on demand and connected to a user-space process to provide
the actual I/O.  This is how things like most interactive network
servers are implemented: something like a `telnetd` daemon runs in
userspace (though possibly as a privileged user, such as `root`) and
implements the TELNET protocol over a TCP connection.  While the TCP
protocol itself is implemented in the kernel, the TELNET protocol is
implemented by `telnetd`.  For interaction with the user, that same
program also allocates a pseudo-terminal device and provides data
read from the TELNET connection to that device, and data read from
that device to the network, but wrapped in the TELNET protocol.  The
user interacts with a shell or other program that takes its input
from the pseudo-terminal device and writes its output to the device,
which is then transfered to `telnetd` for transmission over the
network.  In this way, the pseudo-terminal is like a hinge that data
swings around as it makes its way between the user and the network.
This same basic mechanism was repurposed for the window systems
targeting Unix systems.

As this happened, there was a movement in the Unix community to
standardize system interfaces, ultimately resulting in the POSIX and
various Single Unix Specification standards.  With this, the
terminal interface exposed by the kernel was standardized, the
notion of a "line discipline" formalized, and a programming
interface for handling terminals defined.  We can now safely program
to the
[`termios`](http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/termios.h.html)
interface specified by POSIX and expect it to be implemented
compatibly approximately everywhere.  There are even
[bindings](http://sml-family.org/Basis/posix-tty.html) in the
[SML basis library](http://sml-family.org/Basis/).

While antiquated, this gives us a readily available, proven
abstraction to develop on for our BBS software.  Users log into the
Fat Dragon system via the SSH protocol; the SSH server allocates a
pseudo-terminal for the user's session, and the user interacts with
that via whatever program s/he chooses.  Our BBS software can be
written against the normal terminal facilities, freeing it from
having to know or care about the SSH protocol.  Any other program
the user wishes to run can reuse that same terminal for interaction:
if the user wants to run an editor such as `emacs` or `vi`, it will
simply work in the inspected manner, since the user's session with
the host is just an ordinary login session.

## To Recap...

So what Unix gives us:

* A process model, giving our programs isolation and our system
  concurrency.
* Intrinsic multiple user support, letting us use the existing
  account facilities to support multiple BBS users.
* A large compliment of existing programs implementing useful
  utilities, tools, mailers, pagers, editors, programming language
  interpreters and compilers, revision control programs, etc.
* Native support for networking and remote access.
* A useful terminal abstraction that hides the details of the
  network protocol we use to access the system.

This is the rock we'll build on.

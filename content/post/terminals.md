---
title: "Terminals"
date: 2018-12-20T11:30:59Z
---

I'd like to talk a little bit about terminals, ANSI command
sequences and character sets.

## Terminals

In the early days of computers, before the rise of microcomputers,
most machines were either batch-oriented with no real "console" for
general use, or were timesharing machines that were designed to be
accessed remotely via serial "terminals".

The first terminals were hardcopy: they printed directly onto
scrolls of paper, as opposed to a screen.  These early devices were
repurposed electromechanical machines originally designed as either
typewriters or as teletype terminals, which interpreted simple
binary codes like BAUDOT to display text.  These had been in use for
several decades by the time computers arrived on the scene; you know
those old movies from the 1940s were people receive telegrams and
read them out loud? "SO-AND-SO SAYS ALL OK, STOP. CHECK IN THE MAIL,
STOP. MERRY CHRISTMAS AND HEE-HAW, STOP.": those messages were
transmitted via teletype.

Teletype terminals used for computer applications contained actual
print heads and paper feeding mechanisms; limited functionality for
influencing presentation was available. One could, for example, back
up over printed text and print the same text again, giving a bolded
effect.  But the terminals themselves understood little in the way
of control messages other than advancing the paper feed or returning
the print carriage to the left margin; the so called "carriage
return" (invariably most terminals printed left-to-right scripts).

5-bit BAUDOT, which only had a very small encoding and relied on a
stateful transition to a separate code space to represent more than
32 characters, was quickly replaced by ASCII (the "American Standard
Code for Information interchange"), a 7-bit character encoding that
could represent upper- _and_ lowercase letters as well as numbers
and most common punctuation characters, in addition to a fair number
of control characters.  As an interesting aside, the set of ASCII
control characters clearly indicate that its designers were
interested in representing whole message frames, as it contains
characters for representing unprinted things like "START OF HEADER"
and "END OF MESSAGE."  Characters with ordinal value less than 32
are control characters in ASCII.

Importantly, our early Unix machines were programmed via hardcopy
terminals.  The
[famous picture](https://en.wikipedia.org/wiki/Ken_Thompson#/media/File:Ken_Thompson_(sitting)_and_Dennis_Ritchie_at_PDP-11_(2876612463).jpg)
of Ken Thompson and Dennis Ritchie in front of a PDP-11 shows
Thompson seated in front of a Teletype brand terminal; probably an
ASR-33.

At some point low-cost graphical displays emerged, and manufacturers
started shipping terminals that contained an integrated screen that
could display some number of lines of text; often 24 or 25 (24 lines
of text plus a status line) with a line width of 40 or
[80 columns](http://pub.gajendra.net/2012/09/hollerith_tyranny).

Initially the screen was treated as an infinite paper scroll, thus
repurposing the software written for the earlier and simpler
hardcopy terminals.  But it wasn't long until manufacturers started
putting small microcontrollers into terminals and programming them
to accept commands embedded in a data stream that could do things
like control the terminal's cursor position, set attributes on text
(reversing the fore- and background colors, or bolding text for
example).  These terminals quickly displaced and first-generation of
graphical "dumb" terminals.

Of note, these terminals are usually "character at a time", meaning
that they transmit a character to the host computer after each
keystroke entered by the user; a variation on this theme is "line at
a time", in which the terminal accumulates a line of text in a
buffer and then transmits it to the host.

When "intelligent" graphical terminals came on the scene, each
terminal manufacturer devised a set of commands and an encoding for
those commands specific to their product.  For instance, the command
to set the cursor position to column "y" on row "x" for the IBM 3101
series of terminals is "\<ESC\>Ya,b" where "a = x+32" and
"b = y+32". Note that the offsets allowed "a" and "b" to be
printable ASCII characters.  However, the Wyse-50 uses "\<ESC\>=ab",
where "a" and "b" are defined similarly to the IBM3101.  Curiously,
the DEC VT52 terminal, uses the same command as the IBM 3101, but
the similarity is passing.  Arbitrary differences between
manufacturer's command sets had the obvious drawback that supporting
specific terminal types meant having special programs for different
kinds of terminals.

In the Unix world, this was addressed by the observation that, while
command sets differed between terminal models, the operations
themselves tended to fall into a fixed set of operations: "set
cursor position", "delete a character", "insert a string", etc.  If
the specific command for each operation could be encoded in some
program-independent way, such as into a file, then one could build a
library to retrieve the command for a given operation and emit it to
the terminal.  One would then write programs against that library,
and they would be independent of the underlying terminal.  The
result was the [`termcap`](https://en.wikipedia.org/wiki/Termcap)
library.  Once that existed, a general-purpose toolkit for creating
terminal-oriented programs became an obvious thing to build: the
[`curses`](https://en.wikipedia.org/wiki/Curses_(programming_library))
library was the result.  Uses curses, one could program to
higher-level abstractions such as windows, menus, etc.  This served
Unix admirably throughout the 1970s and into the 1980s.

But few systems took this approach.  The alternative was to build
support for common terminal types into each program, or to build it
into the operating system.  Multics chose the latter approach and
given that system's segmented single-store nature and support for
dynamic linking, the approach worked well: terminal support could be
built into a segment that exposed a generic interface that could be
shared between any number of programs with minimal support from the
operating system.

But other systems didn't support more than a handful of available
terminals and so finally, to address this mess, a committee was
formed to come up with a standard command set: the ECMA-48 standard
of 1978 specified a common set of "escape codes" that could be
implemented by terminal manufacturers; this standard was adopted as
ANSI X3.64 in 1981.  The VT-100 terminal from Digital Equipment
Corporation (DEC) was one of the first to adopt this command set,
and many other manufacturers soon followed.  Of course, some
introduced proprietary extensions and the standard has been updated
over the years.

With the rise of workstations with bitmapped graphical displays,
dedicated serial terminals declined in popularity and they are no
longer in widespread use.  However, the command sequences live on
in applications such as `xterm` and other similar programs.  These
also often support extensions of the earlier standard, such as
256-color support (compared to 16 colors on DOS etc).

As an historical aside, we should note that the terminal market
bifurcated into two camps at one point: on one hand were the sort of
character-at-a-time terminals we have been discussing.  On the other
were block-oriented terminals such as those in the in the IBM 3270
series used on mainframe computers.  These terminals present a
screen to the user with editable fields, but modifying the contents
of those fields is local to the terminal (or more properly, an
ancillary computer called a terminal controller, or sometimes
terminal concentrator).  By pressing a special "submit" key, the
user transmits the entire contents of the screen to the host, which
then extracts the user-provided fields and acts on them.  IBM's VTAM
(Virtual Terminal Access Method) can be used to program these.  We
mention these only for completeness, but will not discuss them
further, as they are largely irrelevant BBS systems.

Anyway, by the time the IBM Personal Computer was introduced in
1981, the ANSI standard had won in the marketplace for terminals.
Although not initially supported by MS-DOS, there was sufficient
demand that Microsoft and others produced a DOS add-on that allowed
the PC to interpret ANSI terminal command sequences and update the
display accordingly, making the PC useable as a terminal.  With the
Color Graphics Adapter in the PC, it could even support 16 colors in
the display.  Other microcomputers similarly adopted ANSI escape
sequences in communications programs (which took on the moniker
"terminal programs" for obvious reasons), and BBS authors took
advantage of this to enhance the visual appeal of their offerings.

So when BBS enthusiasts talk about "ANSI", they are referring to
these escape sequences.  However, that's only part of the story.

## Character Sets and Code Pages

By the time of the microcomputer revolution, the world had more or
less standardized on the 8-bit byte; the IBM PC was no different. It
may surprise some readers that this was not always the case, but
most early computers were word-oriented and had variable length
bytes.  IBM had entered the world of the 8-bit byte with the System
360 series of mainframe computers, and by the time of the IBM PC
they were well-versed in this.  Moreover, they chose ASCII as the
basis of the PC's character set (on the mainframe, EBCDIC was far
more common).

The astute reader will observe, however, that ASCII was a 7-bit
character set, while bytes on the PC were 8 bits wide.  This meant
that the ASCII character set only took _half_ of the available code
space; the other half was effectively empty and could be repurposed
for additional functionality.

The PC designers decided to take advantage of this by programming
the in support for an extended character set that included ASCII in
the lower half, and a set of pseudo-graphical characters and some
characters common in western European languages in the upper half.
This allowed PC programmers to produce semi-graphical programs
simply by writing text to the computer's graphics adapter.  The
specific character set they settled on was designated "IBM Code Page
437," or CP437 which is derived from the page number of IBM's
character set manual: the PC character code was described on page
437, hence "code page 437".  The pseudo-graphical characters were
sometimes referred to as "high ASCII", and the entire character set
was sometimes called "extended ASCII" or "IBM extended ASCII."

Since CP437 retained ASCII as a subset, and ANSI terminal escape
sequences could be be used with ASCII character encodings, and since
the command sequences used by ANSI terminals didn't conflict with
the characters in the high region of the CP437 character set and
since PC-DOS and later MS-DOS supported ANSI sequences, it was
natural that terminal-oriented applications written on and for the
PC would use the extended CP437 character set with ANSI escape
sequences.  BBSes were no exception to this, and indeed, may have
been the exemplar of the style.

Thus, when BBS enthusiasts refer to "ANSI", what they _really_ mean
is the combination of ANSI terminal escape sequences as per ECMA-48
along with the CP437 character set.  This served the BBS community
well throughout the 1980s and into the 1990s, until the widespread
availability of Internet access displaced BBSes.

## Unicode and UTF-8

Of course, the world's languages and their scripts are a lot richer
and far more complex than what is expressible with ASCII and CP437.
So while the BBS world was stuck in that technology, the rest of the
world was eager to move on to something more expressive.  The result
was the Unicode standard, which seeks to define standards for all of
the world's languages.  Given that there are a lot more symbols in
the union of all those languages than are expressible in an 8-bit
byte, the first order of business was to use a wider integer for
representing character data, but this presented a problem: while the
order of bit transmission for 8-bit _octets_ between systems was
universally well-defined, the same was not true for larger data.
Further, there are two competing schools of thought for how to
represent the ordering of bits within wider quanties: _big endian_
where the most-significant bit of an integer has the lowest address,
and _little endian_ where the least-significant bit is at the lowest
address.  Also, for "plain text" using the Latin alphabet, the
wider encoding was less efficient than ASCII.

At Bell Labs, the same group that had invented Unix and C had moved
on to [Plan 9](http://pub.gajendra.net/2016/05/plan9part1) as their
research base.  The team was eager to implement Unicode, but since
Plan 9 was inherently a distributed system, encoding issues were
paramount.  The issue was resolved by Ken Thompson and Rob Pike, who
[invented](http://doc.cat-v.org/bell_labs/utf-8_history) the UTF-8
encoding.

UTF-8 is a byte-oriented encoding where wider Unicode "code points"
are deconstructed into smaller constituent bit values that are
packed into bytes; the high bits of the byte are used to determine
whether one has reached the end of the encoding for a particular
code point.  It is self-synchronizing (meaning that one can find the
beginning of a code point should a stream drop a byte) and
coincident with ASCII in the low 7 bits, so efficient for the common
case of plain text.  Importantly, since it is byte-oriented, it
side-steps the endian issue entirely.

As the Unicode standard has grown to encode more and more characters
from more and more alphabets and scripts, UTF-8 has been extended to
support larger integer sizes for handling code points (called
"Runes" in Plan 9's nomenclature).  It supplanted ASCII and is now
the standard encoding for text interchange on the Internet.

## Back to the BBS

The state of the world today is this:

* CP437 is dead.  It is entirely a legacy format, and has been
  replaced by UTF-8 and Unicode.
* Serial terminals are no more in widespread user use, but ANSI
  command sequences for terminal are interpreted by software for
  text-oriented applications.
* Terminal applications support basic ANSI sequences but also
  often extensions, such as 256 colors.  `xterm` seems to be
  ubiquitous.

What does this mean for us?  If we want to implement a modern,
Unix-based BBS, we should target `xterm` as the display driver, and
we should use UTF-8.  In particular, we should not feel compelled to
maintain compability with DOS-based technologies and standards.  We
should feel free to leave the world of CP437 and the limitations of
`ANSI.SYS` behind.

So that's what we'll do.

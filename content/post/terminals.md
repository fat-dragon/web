---
title: "Terminals"
date: 2018-12-20T11:30:59Z
---

I'd like to talk a little bit about terminals, ANSI command
sequences and character sets.

## Terminals

In the early days of computers, before the rise of microcomputers,
most machines were either batch-oriented with no real "console" for
general use.  Later, most computers were timesharing machines that
were designed to be accessed remotely via serial "terminals".

The first terminals were hardcopy: they printed directly onto
scrolls of paper as opposed to a screen.  These early devices were
repurposed electromechanical machines originally designed as either
typewriters or as teletype terminals and they interpreted simple
binary codes like BAUDOT to display text.  These had been in use for
several decades by the time computers arrived on the scene; you know
those old movies from the 1940s were people receive telegrams and
read them out loud?  "SO-AND-SO SAYS ALL OK STOP CHECK IN THE MAIL
STOP MERRY CHRISTMAS AND HEE-HAW STOP": those messages were
transmitted via teletype.

Teletype terminals used for computer applications contained actual
print heads and paper feeding mechanisms, and thus provided only
limited functionality for influencing presentation.  For example,
one could back up over printed text and print the same text again,
giving a bolded effect.  Or one could back up over printed text and
print underscore characters, giving an underlined effect.  But the
terminals themselves understood little in the way of control
messages other than advancing the paper feed, backing up a single
character at a time or returning the print carriage to the left
margin; the so called "carriage return."  Invariably most terminals
printed the Latin alphabet from left to right.

5-bit BAUDOT had a very small encoding space and could only
represent a handful of symbols.  It relied on a stateful transition
to a separate code space to represent more than 32 characters.  Once
it appeared, ASCII (the "American Standard Code for Information
interchange"), a 7-bit character encoding that could represent
upper- and lowercase letters, numbers and most common punctuation
characters as well as a fair number of control characters, quickly
replaced the earlier encoding.  As an interesting aside, the set of
ASCII control characters imply that its designers were interested in
representing whole message frames, as it contains characters for
representing non-printing things like "START OF HEADER" and "END OF
MESSAGE."  Characters with ordinal value less than 32 are control
characters in ASCII.

Importantly, our early Unix machines were programmed via hardcopy
ASCII terminals.  The
[famous picture](https://en.wikipedia.org/wiki/Ken_Thompson#/media/File:Ken_Thompson_(sitting)_and_Dennis_Ritchie_at_PDP-11_(2876612463).jpg)
of Ken Thompson and Dennis Ritchie in front of a PDP-11 shows
Thompson seated in front of a Teletype brand terminal; probably an
ASR-33.

At some point low-cost graphical displays emerged, and manufacturers
started shipping terminals that contained an integrated screen that
could display some number of lines of text; often 24 or 25 (24 lines
of text plus a status line) with a line width of 40 or
[80 columns](http://pub.gajendra.net/2012/09/hollerith_tyranny).

Initially the screen was treated as an infinite paper scroll and
software written for the earlier and simpler hardcopy terminals was
used without modification.  But it wasn't long until manufacturers
started putting small microcontrollers into terminals and
programming them to react to commands embedded in the data stream
they received from the host.  These commands could, for example, set
the terminal's cursor position, set text attributes like reversing
the fore- and background colors, or bolding text.  These
"intelligent" graphical terminals quickly displaced the
first-generation of "dumb" video terminals, but required software
modifications to take advantage of the new functionality they
provided.  In particular, programs had to explicitly emit the
control sequences understood by the terminal.

Of note these terminals are usually "character at a time", meaning
they transmit a character to the host computer in response to each
of the user's keystrokes; a variation on this theme is "line at a
time" where the terminal accumulates a line of text in a buffer in
the terminal and then transmits it to the host.  These terminals may
either be full-duplex, meaning they can transmit and receive data
to/from the computer simultaneously, or they can be half-duplex so
that they can transmit or receive but not at the same time.  The
terminal may echo characters entered by the user locally, or that
may be handled by the host, which receives characters from the user
and then re-transmits them to the terminal for display; this latter
mode is useful because the host can actively control the user's
display to do things like hiding the character's of the user's
password when entered, or providing word-wrapping functionality if
the user types more than a line's worth of text.

When these intelligent graphical terminals came on the scene, each
manufacturer devised a set of commands and an encoding for those
commands specific to their product.  For instance, the command to
set the cursor position to column "y" on row "x" for the IBM 3101
series of terminals is "\<ESC\>Ya,b" where "a = x+32" and "b = y+32"
and "\<ESC\>" is the ASCII "escape" character (ordinal value 27
decimal).  Note that the offsets allow "a" and "b" to be printable
ASCII characters, since those characters start at ordinal value 32
decimal (which happens to be the space character).  However, the
Wyse-50 uses "\<ESC\>=ab", where "a" and "b" are defined similarly
to the IBM3101 and the "=" is a literal ASCII equals-sign character.
Curiously, the Digital Equipment Corporation (DEC) VT52 terminal
uses the same command as the IBM 3101, but the similarity is
otherwise passing and arbitrary differences between manufacturer's
command sets had the obvious drawback that supporting different
terminal types meant having special program support for each
specific kind of terminal.

In the Unix world, this was addressed by the observation that while
command sets differed between terminal models, the operations
themselves tended to fall into a fixed set of operations: "set
cursor position", "delete a character", "insert a string", etc.  If
the terminal specific command for each operation could be encoded in
some device-independent way, such as a format string, and if those
strings could be stored in a file or entered into a database that
could be quickly queried by terminal model, then one could build a
library to retrieve the command sequence for any given operation and
device and emit the correct device-specific commands to the
terminal.  One would then write programs against that library and
they would be independent of the underlying terminal.  The result
was the [`termcap`](https://en.wikipedia.org/wiki/Termcap) database
and associated libraries.

Once that existed, a general-purpose toolkit for creating
terminal-oriented programs became an obvious thing to build: the
[`curses`](https://en.wikipedia.org/wiki/Curses_(programming_library))
library was the result.  Uses curses, one could program to
higher-level abstractions such as windows, menus, etc.

This served Unix admirably throughout the 1970s and into the 1980s.

Multics chose another approach based on that system's segmented
single-store nature and dynamic linking: support for specific
terminal devices could be built into a segment that implemented a
generic interface that programs where written against.  These
segments could be dynamically loaded into any number of programs
with minimal support from the operating system, allowing those
programs to access the terminal's functionality via the generic
interface.

But few systems took these approaches.  The alternative was to build
support for common terminal types into each program, or into the
operating system, so most systems didn't support more than a handful
of available devices.  To address this mess, a committee was formed
to come up with a standard command set: the ECMA-48 standard of 1978
specified a common set of "escape codes" that could be implemented
by terminal manufacturers and this standard was adopted as ANSI
X3.64 in 1981.  The DEC VT100 was one of the first terminals to
adopt this command set, and many other manufacturers soon followed.
Of course, some introduced proprietary extensions and the standard
has been updated over the years, such as support for multiple colors
(often 16).

Starting with the rise of workstations with bitmapped graphical
displays in the late 1980s, dedicated serial terminals declined in
popularity and they are no longer in widespread use.  However, the
ANSI and VT command sequences live on in applications such as
`xterm` and other similar programs.  These often support extensions
of the earlier standard, such as 256 color support.

As an historical aside I should note that the terminal market
bifurcated into two camps at one point: on one hand were the sort of
character-at-a-time terminals we have been discussing.  On the other
were block-oriented terminals such as those in the in the IBM 3270
series used on mainframe computers.  These terminals present a
screen with editable fields to the user, but modifying the contents
of those fields is local to the terminal (or more properly, an
ancillary computer called a terminal controller, or sometimes
terminal concentrator).  By pressing a special "submit" key, the
user transmits the entire contents of the screen to the host, which
then extracts the user-supplied fields for processing.  IBM's VTAM
(Virtual Terminal Access Method) can be used to program these.  I
mention these only for completeness but will not discuss them
further as they are mostly irrelevant to BBS systems.

Anyway, by the time the IBM Personal Computer was introduced in
1981, the ANSI terminal standard had won in the marketplace.
Although not initially supported by PC-DOS, there was sufficient
demand that Microsoft and others produced DOS add-ons that allowed
the PC to interpret ANSI command sequences and update the display
accordingly, making the PC useable as a terminal; with the Color
Graphics Adapter it could even support 16 colors.  Other
microcomputers similarly adopted ANSI escape sequences in
communications programs (which took on the moniker "terminal
programs" for obvious reasons), and BBS authors took advantage of
this to enhance the visual appeal of their offerings.

So when BBS enthusiasts talk about "ANSI", they are referring to
these escape sequences.  However, that's only part of the story.

## Character Sets and Code Pages

By the time of the microcomputer revolution, the world had more or
less standardized on the 8-bit byte; the IBM PC was no different. It
may surprise some readers that this was not always the case, but
most early computers were word-oriented and had variable length
bytes.  IBM had entered the world of the 8-bit byte with the System
360 series of mainframe computers and were well-versed in
byte-oriented software by the time of the IBM PC.  Moreover, they
chose ASCII as the basis of the PC's character set instead of
EBCDIC, which was standard on their mainframe offerings.

The astute reader will observe, however, that ASCII is a 7-bit
character set, while bytes on the PC were 8 bits wide.  This meant
that the ASCII character set only used _half_ of the available code
space; the other half was effectively empty and could be repurposed
for additional functionality.

The PC designers took advantage of this by programming support for
an extended character set into the display adapter that included
ASCII in the lower half, and a set of pseudo-graphical symbols and
some characters common in western European languages in the upper
half.  This allowed PC programmers to produce semi-graphical
software simply by writing character bytes to the computer's
graphics adapter.  The specific character set they settled on was
designated "IBM Code Page 437" or CP437; the name is derived from
the page number of IBM's character set manual where the set is
documented: the PC character codes were described on page 437, hence
"code page 437".  The extended glyphs were sometimes referred to as
"high ASCII", and the entire character set was sometimes called
"extended ASCII" or "IBM extended ASCII."

Since CP437 retained ASCII as a subset, and ANSI terminal escape
sequences were designed be be used with ASCII character encodings,
and since the command sequences used by ANSI terminals didn't
conflict with the characters in the high region of the CP437
character set and since PC-DOS and later MS-DOS supported ANSI
sequences for display manipulation, it was natural that
terminal-oriented applications written on and for the PC would use
the extended CP437 character set with ANSI escape sequences,
particularly over serial ports.  BBSes were no exception to this,
and indeed, may have been the exemplar of the style.

Thus, when BBS enthusiasts refer to "ANSI", what they _really_ mean
is the combination of ANSI terminal escape sequences as per ECMA-48
along with the CP437 character set.  This served the BBS community
well throughout the 1980s and into the 1990s until the widespread
availability of Internet access displaced BBSes.

## Unicode and UTF-8

Of course, the world's languages and their scripts are a lot richer
and far more complex than what is representable with ASCII and
CP437. So while the BBS world retained that technology, the industry
was eager to move on to something more expressive.

To this end, multiple character encodings for specific languages
gained and lost popularity during the 80s and 90s.  For example, ISO
8859-1 ("ISO-Latin1"), which covered most of western European
languages, used the Latin alphabet and contained ASCII as a subset
was popular in the Unix world.  But none of these encodings were
sufficient and eventually the Unicode standard, which seeks to
define a standard encoding for all languages, was created and rose
to prominence, displacing all earlier encodings.

The fundamental problem was that these earlier encodings were based
around bytes; given that there are a lot more symbols in the union
of all languages than are expressible in an 8-bit byte, the first
order of business for Unicode was to use a wider integer type for
representing character data.  But this presented a problem: while
the order of bit transmission for 8-bit _octets_ between systems was
universally well-defined, the same was not true for larger data.
Further, there are two competing schools of thought for how to
represent the ordering of bits within wider quantities: _big endian_
where the most-significant bit of an integer has the lowest address,
and _little endian_ where the least-significant bit is at the lowest
address (incidentally, the "endian" terminology derives from
_Gulliver's Travels_, where two groups of Lilliputians warred over
which end of an egg to start eating from).  Also, for "plain text"
using the Latin alphabet, the wider encoding was less efficient than
ASCII.

At Bell Labs, the same group that had invented Unix and C had moved
on to [Plan 9](http://pub.gajendra.net/2016/05/plan9part1) as their
research base.  The team was eager to implement Unicode, but since
Plan 9 was inherently a distributed system, encoding issues were
paramount.  The matter was resolved by Ken Thompson and Rob Pike,
who [invented](http://doc.cat-v.org/bell_labs/utf-8_history) the
UTF-8 encoding.

UTF-8 is a byte-oriented encoding where wider Unicode "code points"
are deconstructed into smaller bit values that are packed into
bytes; the high bits of the byte are used to determine whether one
has reached the end of the encoding for a particular code point.  It
is self-synchronizing (meaning that one can find the beginning of a
code point should a stream drop a byte) and coincident with ASCII in
the low 7 bits, so efficient for the common case of plain text.
Importantly, since it is byte-oriented, it side-steps the endian
issue entirely.

As the Unicode standard has grown to encode more and more characters
from more and more alphabets and scripts, UTF-8 has been extended to
support larger integer sizes for handling code points (called
"Runes" in Plan 9's nomenclature).  It supplanted ASCII and is now
the standard encoding for text interchange on the Internet.
Importantly, Unicode contains every glyph from the CP437 character
set.

## Back to the BBS

The state of the world today is this:

* CP437 is dead.  It is entirely a legacy format, and has been
  replaced by UTF-8 and Unicode.  Everything in CP437 has a
  corresponding Unicode code point.  Most terminal programs support
  UTF-8 encoding of Unicode text.  Fonts, such as
  [unscii](http://pelulamu.net/unscii/) exist for supporting the
  sort of "character cell" art common on BBSes.
* Serial terminals are no longer in widespread user use, but ANSI
  command sequences for terminal are interpreted by software for
  presentation of text-oriented applications.
* Terminal applications support basic ANSI sequences but also
  often extensions, such as 256 colors.  `xterm` seems to be
  ubiquitous and is based on the ANSI standard.

What does this mean for us?  If I want to implement a modern,
Unix-based BBS, I will target `xterm` and things compatible with
it as the display driver, and I will use UTF-8 for text encoding.

In particular, I do not feel compelled to maintain compatibility
with DOS-based technologies and standards.  I feel completely free
to leave the world of CP437 and the limitations of `ANSI.SYS`
behind.

So that's what I'll do.

---
title: "Goals"
date: 2018-11-30T16:44:27Z
---

I should state some goals up front.

An explicit goal is to **not** do things that made sense for DOS,
dial-ups, and Fido Technology Network (FTN)-style messaging, except
as insofar as they still make sense for modern systems.  Rather, the
point is to deliberately do things differently.

I say this because many of the things that made sense for those
earlier systems were dictated by their limitations, but given that
modern systems don't have the same limitations, there is no reason
to continue being bound by those decisions.  More generally, things
change over time and if we are unwilling to reevaluate decisions in
light of changing environments, we stagnate and die.

Take Fidonet as an example: why would one ever build that when UUCP
existed?  Perhaps Tom Jennings didn't know about UUCP, though I find
that doubtful.  Perhaps he was worried about legal entanglements
with AT&T or didn't have the documentation to build a compatible
implementation, though Tom Dell obviously did it for Waffle.  Maybe
he felt it was too complex or otherwise overkill for the application
domain.  For whatever the reason, it was built.

But I find it somewhat curious that FTN-style messaging is still
used in the BBS world.  Don't get me wrong, I see that it's useful
and folks enjoy working with it, but I don't understand why it
hasn't been replaced with something based on, for example, HTTP and
a syndication format.

One can imagine representing messages in some kind of extensible,
text-based, structured format like JSON or similar, and embedding
those in something like Atom, all distributed via an HTTP server.
Or perhaps one might represent each message as a separate resource
and point to them via RSS 2.0 enclosures, again transferring via
HTTP.

In this architecture, one automatically reaps all of the benefits of
modern web infrastructure: implicit TLS, on-the-fly compression,
web-based authentication, etc, but using whatever specific server or
clients you wanted on either end.  Message transfer is a `GET`
command.  More generally, one leverages Internet-centric
infrastructure and integrates more closely with the web as a whole.
It is not necessarily what you access from a web browser, but still
very much in line with the original vision for the web.

Instead, the BBS world seems to be stuck with legacy file formats
like QWK, JAMM, etc, and protocols like binkp and zmodem.  Zmodem
specifically I don't understand at all: it seems to me that HTTP
over TCP is superior in every way.

As for the DOS thing...

BBS software packages tend to be implemented as monolithic,
hermetically sealed programs with highly captive user interfaces.
They encapsulate a bunch of functionality and build in a number of
protocols for things like mail, file transfer, etc; they provide
their own UI and tools (editors, user management, etc).  User's are
locked into the user interface and can't access anything else on the
computer that hosts the BBS.  I wonder why, and I think it's
actually due to emulating the original Christensen and Suess model;
but that model came about as a result of the primitive nature of the
machine CBBS was developed and ran on.  Truthfully, DOS machines
weren't much better than CP/M in this regardand when all you have is
a single-user, single-tasking program loader in lieu of a real
operating system, of course you're going to lock the user into a box
so s/he can't destroy your filesystem (let's be honest: back in the
heyday, there were lots of angsty teens trying to mess with people).
If you have no security model or process abstraction you build all
of that stuff yourself into a single program.

But aside from a few retro enthusiasts no one is using systems like
that anymore.  With a multitasking, multiuser operating system and a
well-understood security system, it's possible to distribute the BBS
functionality between multiple programs.  Why implement your own
user management when the system supports that natively?  If you use
the system's support for multiple users, then you can use things
like the system's support for groups and ACLs for access control to
different parts of the "board".  Similarly, once BBS users are just
normal user accounts, you can use the system provided SMTP server:
an annoying bit of code you no longer have to write yourself.  Why
bother writing code for serving the TELNET protocol when a perfectly
decent telnet server with decades of development and testing support
comes with the operating system or is trivially installable as a
third-party add-on?  Even better, why use TELNET at all?  It is
antiquated and fabulously insecure.  Sure it's easy to implement,
but since you don't have to implement it, just use the SSH server
that comes with the system: it's secure and it also gives you
support for secure file transfer via SFTP.  "File bases" are just
directories and text editors, games, etc, all similarly just
programs installed normally.  And if a user wants to pop out of the
menuized BBS environment, well...we've secured the base system well
enough to allow that: why not?  Go for it; hit the shell, write a
program, type an email message to another user, edit a document.
The BBS can be so much more than "just" a BBS.

Another BBS quirk that has not aged gracefully into the modern era:
BBS time limits.  If we're accessing this thing via TCP connections
multiplexed over a packet-switched network via a broadband
connection, mediated by a pseudo-terminal abstraction synthesized by
the host machine's operating system, then who cares how long a user
is connected for?  Stay logged into the Fat Dragon for days or weeks
if you like; I do. Honestly, I wish users _would_ do that, because
then it feels like more of a community.

Similarly with idle timeouts.  It is not taking up much in the way
of resources for a user to maintain an idle shell.  Go for it. It
will just be there when you want to start doing something.

Also, why a limit on the number of concurrently connected "lines"
for users?  The kernel can support hundreds if not thousands of
pseudo-TTYs; we used to regularly have 70 simultaneous interactive
users on grex.org.  SDF sees that kind of traffic frequently.  The
more the merrier.

All of these things made sense in the dialup days on microcomputers,
when you had to build the infrastructure yourself and we didn't
multiplex multiple users onto a single serial port since those were
connected to switched circuit telephone lines anyway.  All of these
were built to accommodate limitations in the environment, but we
don't live in that world anymore, so why does our software?

Also, I kind of feel like the DOS-centric BBS thing has been done,
and I want to explore doing things a different way than the CBBS
model.  Or, perhaps more accurately, I want to explore taking the
sorts of things we did on mainframes and Unix/VMS networks
contemporaneously with BBSes and bring them to a new audience. Where
much of that work was done behind closed doors as those systems cost
real money, so were mostly the domain of large organizations with
deep pockets, I want to open it up to a larger audience.

I suspect that if Christensen and Suess had been able to start with
a PDP-11/20 or Prime 100 or something, the BBS world would have
looked very different.

What I want to build is something that revisits the base assumptions
about what a BBS is and how it works.  It may not be for everyone;
our public access Unix machines attracted a certain "type" of user
for a long while and those users had very little overlap with the
BBS community.  What I think was missing was a bridge between the
more technically advanced public access Unix machines the mainstream
BBS world, where users expect certain things but are missing out on
the advanced capabilities of more sophisticated systems.

I want to explore building that bridge.

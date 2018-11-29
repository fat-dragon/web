---
title: "Prototypes"
date: 2018-11-28T02:44:53Z
---

So we've gone into context a little bit; now let's turn to
technology.  I suspect from here on out I'll try and alternate
technical posts with other sorts of free-form content.

We'd like to gain some experience with the BBS concept, and the best
way to do that is to prototype something.  So let's start to talk
about prototypes.

Ultimately, our goal is to build a BBS: but what exactly _is_ a BBS
anyway?  We have the model from Christensen and Suess, which seems
to be the mold for all systems.  But we'd like this to be more
sophisticated, if not modern, which means reusing pieces of a larger
system where appropriate.

The DOS-based BBS systems all baked in a lot of functionality, but
why?  Well, because they had to.

Recall that CBBS was menu-driven, in part to keep the average user
from directly accessing CP/M.  Since CP/M wasn't designed for
multiple concurrent users, and since access to the BBS was anonymous
for all intents and purposes, trapping the user in a captive
interface made sense to protect the system itself.  Indeed, the BBS
supported so much functionality: user management and access control;
user interface management; messaging; file service; access to games;
editors; pagers; and etc, etc, etc.  These were things that the
underlying system just didn't provide.

But we're not using CP/M, or MS-DOS, or even Windows.  We're
starting from a timesharing operating system that was designed from
the start to be multi-user and multi-tasking.  Unix was forged in
the fires of university computing departments to protect the system
from student excess; while not perfect, many of the pieces in the
form of multiuser support, accounting, protection, isolation of the
system from normal users, etc, are already there.  Further, we have
a process model and a filesystem backing up much of what we do.  We
have high-quality libraries for many aspects of development.  Most
of what we need is already there: pagers, mailers, development
tools, user management tools, and so on.  We can use a local USENET
server or notesfiles for messaging.  We can build a menu to provide
a simple and convenient user interface.  Finally, the system is open
source and freely available.  Therefore, let's prototype on Unix.

So if the system provides us with so much already, what do we really
need?

Well, we need a way to create accounts and populate the various
system databases with user information.  We need to put the
messaging system we choose into play, and finally we need some sort
of user interface to tie all these things together: a menu system is
just fine, provided we can escape from it when we want.

## Introducing the Fat Dragon

The Fat Dragon is a 64-bit virtual private server running
OpenBSD/amd64.

We chose OpenBSD because of its security reputation, simplicity of
use and administration, and because of the current descendants of
Unix, it arguably "feels" closest to the historical system.

Accounts on the Fat Dragon are just normal Unix user accounts; this
limits the namespace of login names to those obeying standard Unix
conventions (lower-case letters, numbers and the underscore only;
login names must start with a letter) but in exchange for that small
conceit the user has full access to the Unix shell and all of the
development utilities, programs, etc, available on the host system.
We use a local Kerberos server for authentication: creating a Fat
Dragon account automatically enters a principle for that user into
the Kerberos database on the KDC.

We'll build a messaging and menu system for our BBS, but that's
about it: everything else is covered so well by other programs.

File service and transfer? Nope. SFTP sitting on top of an SSH
service is superior in every way to XMODEM, YMODEM, ZMODEM and
Kermit.  We won't bother with that stuff.  The filesystem gives us
everything we need to have an active "file base" abstraction.  But
really, do we even want that?  The web does a much better job tying
all of that stuff together.

User management?  Nope.  Unix does that for us, as we mentioned.

A user interface toolkit?  Nope.  The TTY driver does most of that
for us, and where it doesn't we have ncurses.  Actually, this is a
bit of a lie; our menu system probably has to care, but only a
little bit.

Raw communications interfaces?  Nope.  We won't implement the TELNET
protocol (which is horribly insecure, anyway); we'll use the stock
SSH daemon instead.

Pagers, mailers, editors, etc: all of these come with the system or
can be trivially installed through the ports system.  Since we don't
mind users having normal Unix shell access (we give them accounts,
after all), we're not worried about them escaping from the menu
environment.  Have at it; write a cool program.

This, of course, frees us to provide significant additional
services: programming language interpreters, compilers, source code
revision control tools, editors, etc.  Want to develop a program on
the Fat Dragon?  Enjoy yourself!

What's the machine there for if not to be used?

## So What are we Doing, Again?

So what precisely do we need to build?  Is simply providing a machine
"enough"?  Probably not.

Let's set some ground rules.  There are a few things we're not going
to provide:

* Web service: the Fat Dragon doesn't really have resources for
  this, and besides, there are plenty of places that can do it
  better than we can.  We'll host these posts, but that's it.
* Internet Email: Spam, and the fact that we're giving more or
  less anonymous access to the machine make this just too hard
  to support.  This is a hobby that I'm engaging in purely for
  nostalgia; I don't want to spend too much time supporting it.
  Users looking for an email solution are much better off
  elsewhere.  We'll support email _within_ the system so that
  users can message each other, but not general-purpose Internet
  email.  Of course, users are free to use IMAP/POP and SMTP
  clients on the Fat Dragon if they want; we just won't host the
  server infrastructure.  But if you want to use `mutt` or `nmh`
  or `alpine` with `fetchmail` to read your email?  Have at it.

Okay, so what do we need to do first?

We need to figure out how to give users access to the computer.
Fortunately, it turns out that I already have a solution for
this.  I am involved with grex.org, a public access Unix system.
I wrote the current account creation program that runs there.

How does it work?

It's actually pretty simple: we manually created an account called
`newuser` that runs a special program as it's shell that collects
data from the user and creates an account on the user's behalf.  It
then emails the password to the user.  Want to check it out for
yourself?  Feel free to `ssh newuser@grex.org`.

I took the same basic program and modified it lightly to support
adding principles to a Kerberos KDC for authentication, but
otherwise it works reasonably well on fat-dragon.org.

On Grex, `newuser` doesn't have a password.  The Fat Dragon isn't
quite ready for prime-time yet, though, so it does; get in touch
with me if you'd like an account and I'll get you set up.

What's next?  Menus and messaging, but first, we have to make some
decisions.

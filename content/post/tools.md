---
title: "Tools"
date: 2018-11-29T01:59:26Z
---

It has been said that "a good craftsman never blames his tools" for
shoddy work.

I dislike the "craftsman" analogy for programming; it seems to be
promulgated mostly by consultants with, frankly, little track record
of, you know, actually producing working software.  I've
[written](http://pub.gajendra.net/2012/09/stone_age) a little bit
about this on my personal blog; others have
[written more](https://gigamonkeys.wordpress.com/2009/10/05/coders-unit-testing/).

However, setting aside the craftsmanship analogy, there is still
some truth to the adage: we select tools and if the work we produce
with those tools is of poor quality, we can't really blame the
tools.  But that is predicated on the premise that we professionally
select, curate, and maintain our tools.

So we're at the stage of wanting to prototype some things; we've got
a platform in terms of hardware and operating system; we've got a
programming language and a compiler targeting that platform.  What
else do we need?

Three main things.

First, we need some kind of revision control system.  For better or
worse, the standard has become [`git`](https://git-scm.com), so
that's what we'll use.

Next, we need some way a text editor.  Really, this is a matter of
personal preference but after decades of using `ed`, `vi` and GNU
`emacs` on Unix and
[other](https://www.youtube.com/watch?v=dP1xVpMPn8M)
[editors](https://research.swtch.com/sam.pdf)
[on](https://multicians.org/mepap.html)
[other](https://support.hpe.com/hpsc/doc/public/display?docId=emr_na-c04623261)
[systems](https://www.emacswiki.org/emacs/TecoEmacs), I'd like to
have something a bit more modern, so I have decided to use
[Atom](https://atom.io).  It is cross-platform, it has a very nice
SML mode, and the
[remote-ftp](https://atom.io/packages/remote-ftp)
package lets me securely edit files on the Fat Dragon from a laptop
via the SFTP protocol over SSH.  It has some emacs keybindings for
cursor movement and the like, so it's not completely foreign.  I'm
also using it to edit the Markdown source files that these posts are
generated from.

We need some kind of build system that will let us build separate
source files into a coherent whole program; fortunately, MLton comes
with a build system, the
[ML Basis System](http://mlton.org/MLBasis).

Critically we need some kind of unit testing framework so that we
can create a robust body of automated tests for the code that we
write.
[SMLUnit](http://www.pllab.riec.tohoku.ac.jp/smlsharp/?SMLUnit)
looks very nice, and if we outgrow it there is also
[QCheck/SML](https://contrapunctus.net/league/haques/qcheck/).

That about wraps up our suite of development tools.  Some tools for
website maintenance round out the rest.

This web site is hosted on the Fat Dragon itself.  We're using the
[Hugo](https://gohugo.io) static site generator with the
[base16](https://github.com/htdvisser/hugo-base16-theme) theme,
lightly customized.  This is delivered through the standard OpenBSD
[HTTP server](https://www.openbsd.org/papers/httpd-asiabsdcon2015.pdf).
The documentation is in
[Markdown](https://daringfireball.net/projects/markdown/syntax)
format.

So to summarize, we're using the following tools:

* The server itself is a virtual
  [x86_64](https://en.wikipedia.org/wiki/X86-64)
  host computer.
* [OpenBSD](https://www.openbsd.org) for the operating system.  This
  provides the standard compliment of Unix tools, including file
  manipulation commands, a shell, etc.
* [git](https://git-scm.com) for revision control.
* [Atom](https://atom.io) with
  [remote-ftp](https://atom.io/packages/remote-ftp)
  for text editing of both source code and documentation.
* We're programming in [Standard ML (SML)](http://sml-family.org)
  using the [MLton](http://mlton.org) compiler.
* [SMLUnit](http://www.pllab.riec.tohoku.ac.jp/smlsharp/?SMLUnit)
  for unit testing.
* The [OpenBSD HTTP server](https://www.openbsd.org/papers/httpd-asiabsdcon2015.pdf)
* [Hugo](https://gohugo.io) with the
  [base16](https://github.com/htdvisser/hugo-base16-theme) to
  generate the web site.
* [Markdown](https://daringfireball.net/projects/markdown/syntax)
  for documentation.

And that's our current compliment of tools.  Now let's make some
technology decisions.

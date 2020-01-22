---
title: "Languages"
date: 2018-11-28T16:26:31Z
---

So we're on to building prototypes.  To recap, we're using an x86_64
host running OpenBSD as our target environment.  Having selected
this as our system of choice, we provided a solution for
self-service account creation.  Now we need to think about a
coherent user-interface to tie things together.  While the Unix
shell is powerful, and you can have access to it if you'd like, lots
of users aren't going to like it.  Also, we want to tie disparate
parts of the system together under a single interface, so let's
think about menus and messaging.

But first, what programming language should we use?

We have a lot of choice in this area.

Recall that Christensen's mock-up of CBBS (what we would now call a
prototype) was written in BASIC, but the real program was written in
8080 assembly language.  Traditionally, BBS programs were written in
what was usefully available: the most common choices by the dawn of
the DOS BBS era were Pascal, BASIC and C; the most sophisticated
programmers undoubtedly included some assembler.  At least
[one BBS program](https://www.digiater.nl/openvms/decus/vax89a3/ualr/bbs/)
for the VMS operating system on the VAX was written in VAX FORTRAN
and VMS DCL.

Now, of course, we have a much wider selection to choose from.  We
can use one of those traditional languages; environments for Pascal,
C, BASIC and even FORTRAN are available for OpenBSD/amd64; if we
were masochistic we could even use assembly language.  But none of
those are appealing: they all feel repetitive and programming in
many of them is simply tedious.  Manual memory management is
annoying.

Believe it or not, the `newuser` program for account creation is
written in Perl (with some bits in C), but Perl is declining in
popularity and isn't the most comfortable language to begin with.  I
don't think we'll write a lot more perl code, at least not for
interacting with users.  We may even rewrite `newuser` in something
else eventually.

Contemporary BBS development efforts seem to have shifted to other
languages.  Pascal and C are both still popular, but so are C++, C#,
Python, and even JavaScript (with Node.js).  Ruby and Crystal have
made appearances.

But again, none of those really appeal; Crystal is the most
personally attractive (statically typed and compiled Ruby? Yay!),
but (and this might sound silly) it feels a bit too snazzy for the
application domain.

We'd like a language that's a little obscure and funky, because
that's fun to play around with and we're doing this for _fun_.  I
want something that's not _too_ high-level, but that still provides
higher-level abstractions than C or Pascal; something where we feel
reasonably close to the underlying system but without the tedium of
manual memory management or implementing trivial data structures by
hand.  Something that's common enough that it's not totally foreign
and will be decently supported.

Let's also select something that doesn't require an enormous
runtime, so JVM languages are out.  I personally like compiled
languages with strong static typing, so that rules out dynamic
languages like Ruby, Lua, Python, etc, and sadly, most of the Lisp
family.  We'll exclude optionally typed languages like Julia, too.

We're also going to rule out more academically focused languages
like Haskell and Prolog off the bat.  Either of these would serve,
but they're going to be simply inaccessible to the bulk of BBS
users.  Similarly with the F*'s and idris's of the world.  No J, APL
etc: I'd like to be able to read my own code.

Some amount of functional programming support would be nice.

Some choices pop out:

* Ada
* Go
* Rust
* Something from the ML family
* Objective-C

Any of these is reasonable; all are supported on OpenBSD.  To an
extent the decision is arbitrary.  So for my initial prototyping, I
decided to use Standard ML.

[Standard ML (SML)](http://sml-family.org/)
is a language that grew out of the "meta-language" defined for an
interactive theorem prover developed at Edinburgh in the 1970s.
Indeed, "ML" went on to spawn a family of languages that also
includes the popular OCaml.
[History](http://sml-family.org/history/ML2015-talk.pdf)
documents are available for those who'd like to know more.

Standard ML was an attempt in the research community to define a
commmon language for collaborative work.  However, it turned out
that the language had general purpose appeal; it is used in academic
and production environments.

SML comes with a robust "Basis Library" that provides much
functionality for working with the underlying system.  Lots of great
documentation exists, both online and in books.  It supports
programming in the large through a module system.  It is not purely
functional, so we can easily do things like IO without resorting to
[category theoretical contortions](https://blog.plover.com/prog/burritos.html)
and we can use an imperative subset if we really need to.  It
features strict function argument evaluation, so it's relatively
straight forward to reason about performance and behavior.  Memory
is managed by the runtime and it is garbage collected; strings and
lists are first-class types, and the basis library provides a number
of useful data structures such as trees, hash tables, etc.  It
supports algebraic datatypes and higher order functions, and it is
strongly, statically typed, but uses type inference to get rid of
most of the associated boilerplate.  Destructuring and pattern
matching can be used throughout.  It has formally defined semantics.
No memory leaks, no core dumps and it is pleasantly expressive;
what's not to like?  Finally, we get fast native code execution from
high-quality optimizing compilers that are freely available.

But not in the OpenBSD ports collection.

## Bootstrapping an SML compiler on OpenBSD

OpenBSD doesn't supply an SML compiler anymore.  For years, the
"Standard ML of New Jersey" (or SML/NJ) compiler was in the OpenBSD
ports collection, but it seems that this was removed some time ago.
Oh dear.  We'd like to work with SML, but we don't have a compiler;
what to do?

Simple: get one working.

An investigation of available SML environments gives us some options
to work with:

* We can (re)port SML/NJ, but the generated code apparently
  won't play well with some of OpenBSD's security requirements.
  In particular, it sometimes wants to execute code from
  writable pages.  We can fix that, but it sounds tedious, and
  SML/NJ is a big program.  We want to get to work faster.
* We can port Poly/ML, which is a popular implementation, but
  that looks like a superset of basic SML functionality.
* We can port Moscow ML (mosml), which compiles to bytecode
  and comes with an interpreter, but it doesn't appear to
  support all of the basis library, and it doesn't compile
  to native code.
* We can look at MLton, which is a whole-program optimizing
  compiler.  It has some support for OpenBSD already, and can
  compile to native machine code.  The runtime is written in
  C, while the rest of the system (including the compiler) is
  written in SML itself.

MLton looks the most promising, but we need a way to compile the
compiler.  But as it turns out they support cross-compilation to C;
that is, we compile the SML sources to C code targetted to another
platform. Ah, here we go: I can bootstrap a compiler for OpenBSD
from another machine, in this case, a server running FreeBSD that
supports SML/NJ, where I can build a native version of MLton that I
can then use to generate C code from the compiler's SML sources that
I can compile on OpenBSD.  We will cross-compile the compiler to C
from our working FreeBSD machine, then transfer the result to the
Fat Dragon where we will build a native SML compiler.  We will then
use _that_ to recompile the SML compiler natively, and finally use
that second stage compiler to build a third stage compiler that we
will install.  Easy.

Initial generation of C is successful and we can compile the C code
into an executable, but the resulting compiler binary crashes.
Attempts to get a stack back trace from `gdb` (the system debugger)
yield gibberish; what is going on?  The system message buffer
contains entries showing that MLton processes are being killed on
"sp <n> not inside <range>" traps: okay, clearly we're dying due to
some kind of trap, but what does that mean?

Some searching later, we find a
[change](http://openbsd-archive.7691.n7.nabble.com/stack-pointer-checking-td334747.html)
introduced in OpenBSD 6.4 that enforces that, on entry to the
kernel, the userspace hardware stack pointer points into a region of
virtual memory that has been mapped with the `MAP_STACK` flag.  If
not, the process is killed.  Eureka.  So clearly we need to map our
stacks with the `MAP_STACK` flag.  I find the relevant section of
code, modify it, re-generate the C sources for the compiler on the
FreeBSD machine, compile that on OpenBSD, and now we get somewhat
further, but `mllex`, which dies with the same trap.  Trying to get
a backtrace out of `gdb` still gives gibberish results; what's going
on here?  I file an
[issue](https://github.com/MLton/mlton/issues/277)
on github to get some answers.

Recall that on x86_64, the register `%rsp` points to a call stack
and a function is usually called via the `CALL` instruction, which
takes a destination, pushes the address of the instruction
immediately after the `CALL` (the "return address") onto the stack
and jumps to the destination.  Return from the function is
accomplished by executing the `RET` instruction, which pops the
saved return address from the stack and jumps to it, meaning that
the instruction following the call is then executed.

But Matthew Fluet, a MLton developer, got back to me and it turns
out that MLton does not use the hardware call stack or the `CALL`
instruction when running SML code: it only uses it for the runtime
(which is written in C) and for signal delivery (via POSIX
`sigaltstack()`).  For SML code, the stack is managed dynamically
and function calls are via `JMP` instructions.  The rest of the
time, the hardware stack pointer is just a normal register, and on
the register-starved x86 architecture, the MLton compiler will
generate code to use it as such; code might just move a normal
integer like `42` into `%rsp` to perform arithmetic on it, but if
the program happens to enter into the kernel when doing so, it gets
killed because `42` doesn't point into a stack segment.  Oops.  Note
that this also explains why stack backtraces from `gdb` weren't
useful: since SML code doesn't use the hardware call stack, which
GDB uses when doing a backtrace, the stack contents were essentially
random, confusing `gdb`.

However, it turns out that MLton supports Cygwin, which similarly
wants the hardware stack register left alone when using signals;
Cygwin doesn't support `sigaltstack`, so the signal stack is loaded
into `%rsp` and left alone.  Thus, our fix is to modify the logic in
the compiler's code generator so that the register allocator never
allocates `%rsp` when targeting OpenBSD by following that same code
path used for Cygwin and signals.  The fix is a one-line change.

With these two changes in place (map the signal stack with
`MAP_STACK` and don't allocate the stack pointer register for
general use) the compiler works as expected.

Those changes were submitted back upstream and
[committed](https://github.com/MLton/mlton/commit/c0bee8e42d3aa4a70a0bcc709bca8eac6b35cb4a)
to MLton.

With a compiler in place, we can actually start writing code.  But
first, let's make sure we have the right tools handy.

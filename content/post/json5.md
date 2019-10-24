---
title: "Configuration and JSON5"
date: 2019-10-24T19:52:52Z
---

Now we have some basic infrastructure:

*  We know what sort of system we're running on (Unix)
*  We know how to talk to users (the TTY abstraction)
*  We know what programming language we're using (SML)

But we still don't have a BBS.  What do we need?  Things like
menus and so forth, certainly, but these things need to be
configured somehow.  How do we go about doing this?

The canonical answer is to have some sort of _configuration
file_, but this immediately raises a new question: what format
should that file be in, and how should it be encoded?


## Encoding

There are two basic encoding mechanisms we can choose from:

text
:  The file is represented as plain text, and can be manipulated
   via a standard complement of tools.  For example it can be
   modified with a text editor.

binary
:  The file is in some pre-defined format analogous a RAM-based
   record format.  Usually binary files requires special tools
   that understand that format to manipulate.

Of the two, text is slower, as it must be parsed when consumed
but is infinitely more flexible.  However, it is arguably more
dangerous as a maintainer can trivially introduce mistakes when
modifying text files, whereas presumably a dedicated tool could
detect and prevent the introduction of errors; nothing prevents
one from writing a custom tool to maintain a text-based file
though, and of course the BBS software has to properly validate
data read from the configuration file regardless of encoding.

Moreover, the sorts of computers we are likely to run a BBS on
nowdays are swimming in processor capacity and memory: the small
overhead of reading and parsing a text file is more than made up
for by the convenience of having a human-readable and simply
editted configuration file.

In other words, with text we are no worse off in terms of
ensuring data integrity than we are with binary, and the tiny
performance difference is irrelevant on modern hardware.

So text it is.

## Format

Having decided to use a text file (or a set of text files) for
configuration, we now have to decide what format we want to use
for the data in that file.

There are many choices here, and some common examples include the
venerable INI format, that was popular on MS-DOS and Windows.  XML
was popular around the turn of the century.  Lately, JSON has
captured the popular imagination.

The historical alternative in the Unix world has been to invent an
ersatz file format specific to each application.  That's certainly
possible, but one wonders: what's the point?  There are perfectly
good existing formats to choose from rather than inventing something
new.

For our chosen format, we'd like the following properties:

*  A structured format
*  Easy to parse
*  Easy to read
*  Easy to modify
*  Support for comments

INI, while easy enough to read and parse, isn't particularly
structured.  It really only supports one level of "sectioning" in
a file, with simple key/value pairs in each section.  One could
abuse the key/value format to embed pseudo-structured data in an
INI file, but that feels like a hack.

XML is certainly structured and it is not too difficult to parse
(one of its design goals was that a graduate student be able to
write an XML parser in a day or so), but it's not particularly easy
to read or maintain.

JSON is nicely structured, but the use of string literals as keys
are painful to read and modify and lack of comments makes it less
than ideal.

Fortunately, there is JSON extension called
[JSON5](https://json5.org), which bills itself as "JSON for humans."
This allows us to use arbitrary ECMAscript identifiers as keys, has
comments, and some minor maintenance conveniences like allowing
commas after the last element in array and object items.  It's also
fairly easy to parse.

JSON5 fits select JSON5 as our configuration format.

## Building a JSON5 Parser

Having selected a format, we must now parse it.  Unfortunately,
there does not seem to exist a pre-existing parser for JSON5 in
SML, but writing one isn't that hard.

A full treatment of parsing is beyond the scope of this document,
but roughly parsing a structured file requires reading the contents
of the file into strings, apply a lexical analyzer to those strings
to break them into tokens, and then apply a parser to the resulting
token stream.  One can get progressively fancier about this, with
the parser lazily invoking the lexer whenever it needs another
token, and the lexer in turn reading from the file
character-by-character, etc.  However, our configuration files are
not big, RAM is cheap and it's easy to simply read the entire file
into RAM at one time, create a list of tokens, and then present
these to a recursive descent parser as a fait acompli.

The details of reading the file are lexical analysis are not
particularly interesting, but here is the source code for a complete
JSON5 parser:

```sml
fun parse s =
    let
        fun matchColon (TokColon::ts) = ts
          | matchColon _ = raise Exception "match failed"

        fun parse [] = (Null, [])
          | parse (TokLBrace::ts) = parseObj ts []
          | parse (TokLBracket::ts) = parseArray ts []
          | parse (TokNull::ts) = (Null, ts)
          | parse (TokBool b::ts) = (Boolean b, ts)
          | parse (TokInt i::ts)  = (Int i, ts)
          | parse (TokReal r::ts) = (Real r, ts)
          | parse (TokStr s::ts) = (String s, ts)
          | parse (_::ts) = raise Exception("Unexpected token in parse")
        and parseObj (TokRBrace::ts) kvs = (Object (List.rev kvs), ts)
          | parseObj ts kvs =
            let
                val (key, ts) = parseKey ts
                val ts = matchColon(ts)
                val (value, ts) = parse(ts)
                val kv = (key, value)
            in
                parseObjTail ts (kv::kvs)
            end
        and parseKey (TokStr s::ts) = (KeyString s, ts)
          | parseKey (TokId id::ts) = (KeyIdentifier id, ts)
          | parseKey _ = raise Exception("Bad key")
        and parseObjTail (ts as TokRBrace::tailTs) kvs = parseObj ts kvs
          | parseObjTail (TokComma::ts) kvs = parseObj ts kvs
          | parseObjTail _ _ = raise Exception("Object parse failed")
        and parseArray (TokRBracket::ts) vs = (Array (List.rev vs), ts)
          | parseArray ts vs =
                let val (obj, ts) = parse ts in parseArrayTail ts (obj::vs) end
        and parseArrayTail (ts as TokRBracket::tailTs) vs = parseArray ts vs
          | parseArrayTail (TokComma::ts) vs = parseArray ts vs
          | parseArrayTail _ _ = raise Exception("Array parse failed")

        fun parseSingleObj ts =
            case parse ts of
                (object, []) => object
              | (object, _) => raise Exception("malformed object")
    in
        parseSingleObj (tokenize s)
    end
```

This emits a single JSON object representing our configuration
file; we can query that object with methods to extract the actual
data, possibly packaging this up into an SML record.

For example, suppose we have a configuration file something like
this:

```javascript
{
    //
    // Basic information.
    //
    name: "The Experimental BBS",
    admin: "Dan Cross <cross@fat-dragon.org>",
    host: "fat-dragon.org",
    port: 22,
    //
    // Conferences.
    //
    conferences: [
        {
            name: "local", subdir: "local", title: "Local Conferences",
            moderators: [ "cross" ], acls: [ "all" ],
            conferences: [
                { name: "general", subdir: "general", title: "General Chatter",
                    moderators: [ "cross" ], acls: [ "all" ]
                },
                { name: "hack", subdir: "hack00", title: "Hacking",
                    moderators: [ "cross" ], acls: [ "all" ]
                }
            ]
        }
    ]
}
```

Then we might read it using a sequence something like this:

```sml
type Config =
    { name: string,
      admin: string,
      host: string,
      port: int }

fun readConfig(file: string): Config =
    let
        val cf = TextIO.openIn file
        val txt = TextIO.inputAll cf
        val _ = TextIO.closeIn cf
        val json5Config = JSON5.parse txt
        val config: Config =
            { name = (JSON5.idToString "name" json5Config),
              admin = (JSON5.idToString "admin" json5Config),
              host = (JSON5.idToString "host" json5Config),
              port = (JSON5.idToInt "port" json5Config) }
        handle JSON5.Exception e => (print (e ^ "\n"); raise JSON5.Exception e)
    in
        config
    end

fun main() =
    let
        val config = readConfig("config.json5")
    in
        print ("name:  " ^ (#name config) ^ "\n");
        print ("admin: " ^ (#admin config) ^ "\n");
        print ("host:  " ^ (#host config) ^ "\n");
        print ("port:  " ^ Int.toString (#port config) ^ "\n");
    end

val _ = main()
```

If we compile and run this on the Fat Dragon, we see something
like the following:

```
name:  The Experimental BBS
admin: Dan Cross <cross@fat-dragon.org>
host:  fat-dragon.org
port:  22
```

And so now we have a configuration file format and a means to
read it.  Next, we'll talk about screen handling, and maybe some
actual computer science.

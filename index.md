---
layout: default
---

# Remote Services

> The remote services library is a complete abstraction of client ↔ server
> communication.  
> It is basically a
> [remote procedure call](http://en.wikipedia.org/wiki/Remote_procedure_call)
> implementation in dart.


## Introduction

When you're building web applications you nearly always have different parts of
your application communicating with one another (for example: the browser
requesting data from the server via AJAX).

Most commonly this communication is done by creating a HTTP server, defining an
API and sending JSON data.

There are multiple things wrong with this approach:

1. You need to think of your API *very carefully*. Changing your URL from
    `/user/create` to `/users/create` will break all clients **without them even
    knowing** until they experience the error.
2. The same goes for your JSON data. Consumers of your API will need to look at
    your documentation to know exactly what data will be submitted, and if you
    change something they will never know. A simple typo like `isDeleted` instead
    of `is_deleted` can potentially result in a broken app and it's very hard to
    find that error.
3. *Every client and server* needs to implement the communication (writing the AJAX call,
    handling errors, etc...).

> The *Remote Services library* solves all those problems.


## Dart

*Stop using JavaScript!* Don't get me wrong: I like JavaScript. I wrote two extremely
known libraries ([dropzone](http://www.dropzonejs.com/) and
[opentip](http://www.opentip.org/)), a few lesser known libraries
([mongo-rest](https://github.com/enyo/mongo-rest) and
[node-tvdb](https://github.com/enyo/node-tvdb)) and countless servers and client
libraries for personal and professional use.

But when you work on bigger projects, where stability is an issue and when you
start working with bigger teams, using a language without types is just
incredibly painful! If you don't know a library well, you need to check the
documentation all the time (if there is one up to date), and spotting errors can
be very difficult.

So don't waste any more time and *check out dart*. Once you've starting working
with it for a while, you really start to wonder how you were able to put up with
JavaScript when you have to work on some JS code again. Just clicking on a function
name and getting the declaration, having the dart editor tell you the second you
type a function name what parameters it accepts is just incredibly crucial to a
productive and stable workflow.

## Protocol buffers

Instead of using JSON, which is basically just a Map without any type or
structure information whatsoever, the *remote services* library uses
[protocol buffers](http://en.wikipedia.org/wiki/Protocol_Buffers).
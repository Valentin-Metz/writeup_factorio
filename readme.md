# Abstract

In September 2023 we found a buffer overflow vulnerability in Factorio.
This vulnerability allows for arbitrary code execution when loading or previewing a modified save file.
A fix has been released with version 1.1.94.

# Factorio

[Factorio](https://factorio.com/) is a factory automation game.

It's very popular among computer science students,
as it's the best parts of programming without any of the boring/exhausting parts.

## Finding the bug

Opening factorio in IDA to reverse engineer was just a fun project, but since
I am very interested in security understanding the parsers was a first goal.
Most of it seemed pretty sane and uses C++ dynamic sized types.

There's not really a lot to say about how to reverse engineer a game, you open
it in a disassembler, attach a debugger, find resources online and just piece
stuff together piece by piece. At this point a big thanks to the developers for
giving us debug symbols, however they could at least enable PIE.

There's not really a lot to say, except that following strings and function names
into level loading code until I found the part which loads property-trees from 
maps, again it seems sane in that it always tries to allocate enough memory for 
the entire data part.

![Screenshot of IDA with the line containing the bug being highlighted](/imgs/bug.png)

## The bug

The bug lies within the way this number of bytes to allocate is computed. Because
the number is cast to a 32-bit integer we can enter a number such that this value
gets too small (and thus we will be able to overwrite a whole bunch of stuff on the heap)

Namely this will always give us an overflow of (a multiple of) 4gb, sadly this means that
the mapfile also needs to be this big, else the mapdeserializer will complain that
there simply is not enough data to even attempt deserialization.

There also exists a small piece to implement a custom operator new[] in this C++
project, which first checks if the size to allocate is zero, if yes it will set it to 1
(e.g. no allocation will return nullptr), then it will go into a while true to try
and allocate with malloc and then the `std::get_new_handler` way.

![Screenshot of the new handler in IDA](/imgs/newhandler.png)

## The exploit


## Reporting process

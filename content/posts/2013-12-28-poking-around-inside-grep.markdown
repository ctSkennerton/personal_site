---
title: "Poking around inside grep"
date: 2013-12-27
tags: ['bioinformatics', 'grep']
---

Playing around with the grep source code to make it output fasta/fastq records.
Check out the code [here](https://github.com/ctSkennerton/fagrep).
<!-- more -->

I'm quite interested in string searching algorithms as I've written a 
program, [crass](http://ctskennerton.github.io/crass), which uses a few of them
to search for CRISPR elements.  Crass is pretty fast, but I want it to be faster,
specifically there is one point in crass where it searches for exact matches to many
thousands of patterns.
In a [previous post](/2013/10/28/testing-out-seqans-multipattern-search-implementations/)
I tried out a number of different 'multi-pattern matchers' from [seqan](http://seqan.de)
and was fairly unimpressed with their speed.  In this testing though I did not try
out the most widely used implementations of multi-pattern matching: GNU grep (using `-Ff` options).  I don't have any graphs to show, but it was faster, like **a lot** faster.  

It wasn't a fair fight though, as grep works very differently to my test program
fxtract.  There is a great mailing list [post](http://lists.freebsd.org/pipermail/freebsd-current/2010-August/019310.html)
by one of the original grep authors on ways to speed things up.  It basically 
comes down to I/O (not that the search algorithms aren't highly optimised themselves),
fast input from a file and not copying the data in memory is the key.  Grep doesn't
try to parse the file instead it just loads it into a big buffer in memory and searches
the whole thing, if it finds a match then it figures out the boundaries of the line
the match is on and prints it out.  This is in contrast to programs like fxtract or
crass that parse the file first to get the individual portions of each record before
the search is performed.

This got me thinking that I could drastically speed crass up if I switched over
to the grep way of processing files.  I was a little worried though that determining
the boundaries of a fasta/fastq record from an anonymous buffer might be a bit tricky
so before I modified crass I chose to modify grep so that it would output
fasta or fastq records.

The printing functions in grep live in `main.c` and start with the function `grep`, which
in-turn calls `grepbuf`, which in-turn calls `prtext` etc.  The code is surprisingly simple,
`grep` reads from the file and fills a buffer; `grepbuf` executes
one of the search functions on that buffer; if a match is found, a pointer to the first
character in the line the match was found is returned; and then the printing functions
take over.  The printing functions get a pointer to the start of the line and the end 
of the line of the match and pass that through to `fwrite`.  Everything is handled using
pointer arithmetic for determining the start and end of where to print.

This is great since it's easy to change the pointer to the beginning a end of a record, rather
than a line.  So that just left the logic for me to write in to find the limits of
a record.  Below is a code snippet from grep where I've added in the logic.  Fasta is
easy to implement as the `>` symbol is generally unique.  Fastq on the other hand takes
a bit more work, since the `@` symbol can also be found in the quality string.

```c
char const *b = p + match_offset;  /*pointer to beginning of matching line*/
char const *endp = b + match_size; /*pointer to end of matching line*/
/* Avoid matching the empty line at the end of the buffer. */
if (b == lim)
  break;
if(fasta_input)
  {
    /*find the beginning of the record*/
    while(b > p && b[0] != '>') --b;
    /*find the end of the record*/
    while(endp < lim && endp[0] != '>') ++endp;
  }
if(fastq_input)
  {
    /*find the beginning of the record*/
    while(b >= beg)
      { 
        if(b[0] == '@')
          {
            if(b - 2 <= beg)
              /*can't go back any further therefore must be start of record*/
              break;

            if (b[-1] == '\n' && b[-2] != '+')
              /*@ symbol at beginning of line but not the first in the quality */
              break;
          }
      --b;
    }
    endp = b;
    int newline_count;
    for(newline_count = 0; newline_count <4; ++newline_count)
      {
        /*find the end of the record*/
        while(endp < lim && endp[0] != '\n') 
          ++endp;
    
        ++endp;
      }
  }
```

The fastq format parsing has fairly obvious corner cases since with this
code there can be no text on the 'comment' line and the whole 
record must be of four lines.  This version of fastq is the recommended formatting from the 
official [fastq publication](http://nar.oxfordjournals.org/content/38/6/1767.full),
which seems to have been adopted by Illumina and others, so hopefully this simple
parsing will work most of the time. 

It's been a heap of fun looking at the way this very mature piece of software works
and I've gotten a usable tool out of it.  Now it's onto the main event of 
ripping out parts of the source code that I want for crass
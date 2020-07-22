---
title: "My First Useful Script"
date: 2014-06-23
tags: ['bioinformatics']
---
Can you remember the first useful thing that you ever coded? I sure can, and I'm
thankful for it every day.
<!-- more -->
I've recently finished writing a little program called [`fxtract`](https://github.com/ctSkennerton/fxtract)
(which I've blogged about [before](/posts/2013/10/28/testing-out-seqans-multipattern-search-implementations/))
that acts like grep but returns whole fasta or fastq records from a file. It's
taken me a very long time to write this thing, primarily cause I'm writing it
in C++ but now that it's pretty much done I'm feeling nostalgic about why I'm
writing it in the first place.

Back in 2010 I was just starting my PhD with [Gene Tyson](http://www.ecogenomic.org/users/gene-tyson)
and learning some bioinformatics on a "toy" dataset (which became my first
[paper](http://www.plosone.org/article/info%3Adoi%2F10.1371%2Fjournal.pone.0020095))
that was a huge 10 Mbp of viral sequence data! At the time I was learning Perl
using this great [online tutorial](http://www.woolfit.net/perl/classindex.html)
but I found that I needed some more "real-world" examples to really get to grips
with the language. I can't fully remember what I was doing but I had a tabular
blast output file and I wanted to get the sequence of some contigs with blast
hits. I jumped at the opportunity to solve this with my nasent coding skills and
so I started writing my first useful bit of code:
[`contig_extractor.pl`](https://github.com/ctSkennerton/scriptShed/blob/master/contig_extractor.pl).

In the beginning it did only what I first wrote it to do: take a blast file and
a fasta file and return some contigs. Over time though it morphed into something
more general - a way of getting some subset of a fasta file using a list of
identifiers. Over the years I added functionality for different file formats and
allowed searching using regular expressions.
This one piece of code eventually disseminated throughout the whole lab group
as new people came in and needed to solve the same problems. I think that the
introduction to github for many of the PhD students who came after me
was downloading my random collection of scripts just so they
could get their hands on `contig_extractor.pl`.

Of course every bioinformatician has probably written the same piece of code
to solve the same problem. If I had been smart enough to look I'm sure
that I would have found one, but using a script doesn't quite give the same
level of satisfaction as writing it yourself (and finding that it's useful by all your
colleages).

There came a time though when `contig_extractor.pl` reached it's limit. I was
trying to extract a few thousand records from a fastq file with 100 million and
Perl just wasn't cutting it anymore. And so `fxtract` was born to do all the good
things from `contig_extractor.pl` just a heap faster.The sun is setting on my
first useful piece of code, at the moment only my finger memory and force of habit
keeps it being used. But when I think about it, it's been an amazing four year run for something that I
probably thought was going to be a one-off script, hopefully `fxtract` can get
as much love from me as well.

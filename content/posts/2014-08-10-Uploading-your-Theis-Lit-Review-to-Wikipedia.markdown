---
title: "Uploading your Theis Literature Review to Wikipedia"
date: 2014-11-12
tags: ['bioinformatics']
---
Every PhD student can contribute to open science by uploading their thesis
literature review onto Wikipedia!
<!-- more -->
My thesis was entitled "Phage-host evolution in a model ecosystem", where I
tracked the evolution of phage genome evolution and the evolution of bacterial
defense mechanisms using metagenomics. When I was writing my thesis I spent a
lot of time writing up the section on
[CRISPRs](http://en.wikipedia.org/wiki/CRISPR), which are a type of bacterial
adaptive immune system. The CRISPR field has exploded in recent times due to its
applications for genome editing which has also meant that there have been
numerous great reviews on the ever expanding primary literature. Despite this
though I found the CRISPR page on wikipedia to be a bit bare (of course now that
I've updated it things look better, but check out the edit history for a look at
what I've added). Since there was already a saturation of reviews on the subject
I decided to go through and add in what I had written for my thesis to
wikipedia. By far the most important **and difficult** part of this is was
formatting the references right. So I made a simple workflow for getting what I
wanted out of my thesis and into wikipedia.

1. First thing, I wrote my thesis in word and used endnote to format my
   references using the style from nucleic acids research (which is a numbered
   style). If you want to use my method you need to have the exact formatting
   style
2. Export the literature review from word as plain text and then removed
   all the parts that aren't needed
3. Put back in some formatting for the headers using
   [wikipedia's markup](https://en.wikipedia.org/wiki/Help:Cheatsheet)
4. Split out the bibliography into a separate file
5. Then use some very dodgy [perl code](http://github.com/ctSkennerton/wikipedia_reference_formatter)
   I wrote to format all of the in text citations into the format that wikipedia
   requires

This perl code takes the text and looks for numbers, or lists of numbers inside
parentheses. For example: (1,2,5), (1-7,14), (1). Then looks up what those
references are in the bibliography, extracting the title of the journal article.
Then it looks up what the pubmed ID is for that paper using
[entrezdirect](http://www.ncbi.nlm.nih.gov/books/NBK179288/) with the title as
the search term. After getting the pubmed IDs for all the cited references it
saves them to a file, so that it can be used later on (without looking up the
pubmed IDs again) and any errors can be fixed manually. Finally all of those in
text citations are replaced with the proper wikipedia markup and your ready to
upload.

There are so many scientific wikipedia pages that are way too brief for their
subject matter and I'm not just talking about the thousands of stubs for
bacterial and archaeal species either. My current work is with methane seep
sediment communities that perform [anaerobic oxidation of
methane](https://en.wikipedia.org/wiki/Anaerobic_oxidation_of_methane) (AOM),
the wikipedia page contains five paragraphs. Looking at the broader topic of
[methanogenesis](https://en.wikipedia.org/wiki/Methanogenesis) there is still
only three short paragraphs on the biochemistry. We know so much more than this,
all that knowledge fills the pages of hundreds of review articles yet when
someone, anyone types anything into google the first match that comes up is
wikipedia. We should make it better cause it's ultimately the most visible
publication on the internet.

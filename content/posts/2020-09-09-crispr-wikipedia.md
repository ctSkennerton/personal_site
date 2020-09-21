---
title: 'Checking back in on CRISPRs'
date: Mon, 07 Sep 2020 20:34:14 -0700
tags: ['CRISPR', 'Wikipedia']
---


As part of my PhD thesis I studied an emerging field of bacterial
adaptive immunity, known as [CRISPR](https://en.wikipedia.org/wiki/CRISPR).
At the time I was interested in tracking this type of immune system
in bacterial communities to track co-evolution between bacteria and
their viruses.{{< sidenote "thesis_link">}}  For anyone interested and brave enough,
[here]() is a link to my thesis. {{< /sidenote >}}  After I finished up
writing, submitting, and ultimately obtaining my PhD I realised that
large portions of the literature review section would never be
published in a scientific journal. I came to the realisation that
if this summary of CRISPR (of which I was very proud of) would ever
be read I needed it to be published in an "alternative" way. 

That way was Wikipedia and one of my very
first blog posts summarized my method for [turning my literature
review into a Wikipedia article](/2014/11/12/uploading-your-theis-literature-review-to-wikipedia/).  Perhaps my biggest motivation
for contributing to the Wikipedia page was how out of date the page
was. My literature review from my thesis had dozens of references
and was many pages long yet much of this new information wasn't in
Wikipedia at all. 

Towards the very end of my PhD the research community began studying
[CRISPR for gene
editing](https://en.wikipedia.org/wiki/CRISPR_gene_editing) which
has resulted in amazing (and [controversial](https://www.nature.com/articles/d41586-019-00673-1))
advances in science using this technique. While CRISPR research
exploded I went off on a different research direction and eventually
left academic research for a position in the biotechnology industry.
Despite not being directly involved with CRISPR research anymore I
still maintain two pieces of software,
[minced](https://github.com/ctSkennerton/minced) and
[crass](https://github.com/ctSkennerton/crass), that are designed
to detect CRISPR arrays in either genomes or metagenomes; and
recently I got interested in how the CRISPR page on Wikipedia had
changed since I made my contributions.

Not surprisingly the number of contributions has increased drastically
from 2014 onwards as CRISPR gene editing got significantly more press
coverage. But what did surprise me was Wikipedia's [inbuilt page statistics showing I'm the second
highest author](https://xtools.wmflabs.org/articleinfo/en.wikipedia.org/CRISPR) in terms of characters contributed.
It's been years since I contributed so my
assumption was that most of what I wrote would have been edited
away. Seeing my name so prominent really filled me with pride
as this contribution is likely my most read piece of scientific
literature I've ever made. Digging into the history of the article's revisions it became clear that my
name is still so high on the list due to the
organic nature of Wikipedia article growth. I could see in the history section
that large parts of the CRISPR page had been split off into the
separate [CRISPR gene editing](https://en.wikipedia.org/wiki/CRISPR_gene_editing) page,
which is itself a huge page; and the large
[cas9](https://en.wikipedia.org/wiki/Cas9) page, which details the
primary enzyme used in CRISPR gene editing. Intuitively this makes sense,
people contribute small amounts to a page that already exists until someone
decides that much of that information is better suited under a different title.

Some elements for my original contribution are still present including the table of
signature CRISPR genes and a [summary figure of the different types
of CRISPR systems](https://en.wikipedia.org/wiki/CRISPR#/media/File:The_Stages_of_CRISPR_immunity.svg). 
Both of these elements are sorely out of date.
I was disappointed to see that the table in particular hadn't been
updated as many of the rows still contained descriptions stating
"function unknown". Surely with the increased interest in CRISPR
all of the proteins would have some more information about them.

Thankfully this assumption was true, much research on CRISPR has
taken place but the information has yet to be disseminated back to
Wikipedia. A quick literature search on Pubmed showed amazing
research on many of the different cas genes. This let me 
change a few "function unknown" rows in the table and let me create a few
new enzyme stubs on Wikipedia for
[cas2](https://en.wikipedia.org/wiki/Cas2),
[cas3](https://en.wikipedia.org/wiki/Cas3), and
[cas4](https://en.wikipedia.org/wiki/Cas4).
I hope that the additional
pages will spurn either myself or others to add information.
{{< sidenote "new_additions" >}}Right after I created the cas3 page another user added in some more information within a day! {{< /sidenote >}}

Looking back at this has kindled my own interest in CRISPR again.
A short summary of the information I found and integrated into Wikipedia
while writing this blog post...

### cas4 
Is an endonuclease that works with cas1 and cas2 to generate spacer sequences. It looks for PAM sequences and processes
the spacer before insertion into the CRISPR array.

### cas5, cas6, cas7
These are essential for generating crispr-RNAs (crRNAs) which are used to target foreign DNA.

### cas10
Binds to the 5'-end of the CRISPR RNA and promotes the oligomerization of other cas proteins creating the interference
complex that destroys foreign DNA/RNA


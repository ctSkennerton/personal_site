---
title: "Genome Bin Completeness and Contamination"
date: 2015-01-29
tags: ['bioinformatics']
---
The question that most people ask when looking at a metagenomic draft genome bin
is: should this gene really be there? The answer is that sometimes it's not easy to know

<!--more-->

I spend a lot of my time looking at genome assemblies. They are almost always
from metagenomic data and usually are from some novel phylogenetic lineage
with few (if any) "close" relatives. Unfortunately the quality of genome
assemblies is such that all of these genomes are in many pieces and due to the
nature of binning, there is always the spectre of some genes not belonging to
the genome that your analyzing. I'm currently a middle author on a
[paper](https://peerj.com/preprints/554/) that attempts to quantify genome
completeness and contamination using single copy marker genes. The idea is that
there are genes that are always found in a single copy per genome, therefore
if you find them in multiple copies then you've got some contamination from
other sources. This is great in principle but the number of single-copy genes
is going to be very small for a large set of genomes, for example there are
about 100 of these genes that are common for all bacteria. Assuming that some
bacterial genome has approximately 3000 genes, then looking at only 100 of them
is about 3% of the gene content. That's quite a small percentage to be making
big statements on how good or bad a genome is.

So here is a question that one of my colleagues posed in a group meeting: if a genome has 10%
contamination does that mean that 1 in 10 genes shouldn't be there?

My answer is: of course not, it means that 1 in 10 of the marker genes are in
multiple copies. But it does get me thinking, how can we tell that 10% of all
genes in the genome aren't from contamination?

One of the observations with
single-copy marker genes is that many of them are
co-located such that getting one erroneous contig can disproportionatly increase
contamination. Similarly all of the genes of a contig are either correctly or
incorrectly binned, rather than every 10th gene in some contig being from a
different genome. If we were to assume a perfect situation where every contaminating
contig contained at least one marker gene then the real value of contamination would be
the total number of genes on those contaminating contigs divided by the total number
of genes in the genome bin. The "real" contamination percent then becomes more
about the size of the contigs rather than the number of marker genes. This is
easy in theory but hard to determine in practice since you need to know which
copy of the marker gene is the contaminant and which is legit.

It's easy to remove
contigs with extra marker genes to lower the contamination numbers but what about
short contigs that contain no markers? I don't think there is a perfect way
to tell if they belong to a genome or not but one approach has been to look for
paired reads that link these contigs to larger ones in the assembly even if they
are not contiguous. Here again there are questions raised, if there are reads that
link the contig to the bigger assembly why is it separate, why hasn't the assembler
joined the contigs together? The magic behind genome assemblers creates many
interesting outcomes that I can never explain and as such there is no general rule
for believing paired-end links or not. Honestly, I usually go with my gut
(very unscientific) and generally only bother to go further in depth if there are
genes that are important to the story I'm trying to write.

The golden rule of interpreting genome bin assemblies is that
the bigger the contig the more confident that you become in assigning
them to a draft genome.
(Of course the answer is just getting a better assembly :P).

With these observations and caveats in mind, what value is there in placing a
percent completeness or contamination based on marker genes if it doesn't really
relate to the total gene content?

I think there is a lot of value but not when the numbers are taken literally.
Marker genes are an approximation and
a way of sorting genomes that are worth a closer look. From experience I can
tell you that a genome that is
"90% complete, 5% contamination" looks a lot better than one that is
"50% complete, 30% contamination". I take these numbers to be a relative score
along with other factors like the total number of contigs (less is better) and the
total number of bases (does it look like it's about the size of a whole genome)
to point me in the right direction of what I should work on first.

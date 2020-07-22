---
title: "Genome Scaffolders Suck"
date: 2013-12-11
tags: ['metagenomics', 'bioinformatics']
---

Experiences using a variety of contig scaffolding tools. It was not a good experience. 
<!-- more -->

Recently in our lab we've been getting some Illumina mate-pair data to
improve some metagenomic assemblies.  The sequencing has been going
well and we've been generating a good number of mate-pairs without too much
duplication, but we've had quite a bit of trouble with the bioinformatic part
of actually using this data to improve the assemblies.  There are a number of
software tools available to link contigs together after an assembly has been 
done, however many assume that you are scaffolding a genome not a *meta*genome.  
One of my colleagues suggested that, as a group, we perform a comparison of these 
programs to determine which is the best one to suit our needs.

We decided to test out [sspace](http://www.ncbi.nlm.nih.gov/pubmed/21149342),
[grass](http://www.ncbi.nlm.nih.gov/pubmed/22492642),
[scarpa](http://www.ncbi.nlm.nih.gov/pubmed/23274213),
[opera](http://www.ncbi.nlm.nih.gov/pubmed/21929371),
[mip](http://www.ncbi.nlm.nih.gov/pubmed/21998153) and 
[bambus2](http://www.ncbi.nlm.nih.gov/pubmed/21926123).
From these six programs, only bambus2 was designed for metagenomes.
To test how well genome scaffolders could transition into 
metagenome scaffolders we took one of our metagenomes and randomly broke
up 1000 contigs formed only using paired-end data into two pieces and 
attempted to use some shiny new mate-pair data to put the pieces back
together again.  Little did we know the world of pain that we were about
to step into...

The following post will not talk about which algorithm was better or even
really what results each of these programs gave.  No, I'm going to talk
about why using all these programs sucked.


## The war of attrition
### grass
We didn't even get *grass* installed; it required a particular library
distributed by IBM, which while free for academic users, required you to
give over some personal details, which we decided was something that we
didn't want to do. 

One down, five to go...
### mip
Oh where do I start with *mip*! *Mip* assumes that you're using SOLiD
data, which means that it expects that the orientation of the reads are
forward-forward.  Since Illumina's mate-pairs are reverse-forward I
needed to reverse complement the first read, making a duplicate file.  But
actually I needed to modify the second read file as well because *mip*
needs to have the read identifiers ending in "_R3" or "_F3" (WTF!), just
like they would be for a SOLiD run.  After this, I could then run any short-read
mapper to produce a SAM file for further processing.  But wait, you have
to map each read separately (i.e. not as pairs) and then pass the SAM
file through two preprocessing shell scripts.  The first of these shell
scripts, the opaquely named `merge.sh` simply takes the corresponding
SAM alignment record from the first and second reads and concatenates
the lines!  Why this needed to be a separate script and not just part of
the software proper is beyond me. Both of these preprocessing scripts created new files that
were essentially duplicates of the original SAM files made by the
short-read mapper, which I truly find annoying, especially since our
server is currently short on storage space.

When I ran *mip* proper, it threw an error.  That error may have been
easy for me to fix, it might have been something silly that I had put
into *mip's* configuration file, I don't know.  At this point I was pissed
about the whole process and never tried to run the program again.

Two down, four to go...
### opera 
It segfaulted on the test data that came with the software.

Three down, three to go...
### bambus2
I had hope for bambus, and this was the scaffolder which I put the most
amount of effort to get it to work.  The problem with bambus is
that it is part of the [AMOS](http://sourceforge.net/apps/mediawiki/amos/index.php?title=AMOS)
software package and the problem with AMOS is that it is fairly old
and uses it's own file format.  I have no doubt that the authors of AMOS
were hoping that their file format would catch on and that many tools
would plug into it, but alas the AMOS file format did not catch on,
instead we have SAM files.  So I set about looking for a SAM to AMOS
file converter, and I found [one](http://sourceforge.net/p/amos/code/ci/master/tree/src/Converters/samtoafg.pl) 
in the AMOS package.  It was a nice little perl script that
unfortunately didn't work (once again threw an error).  

I decided to write my own converter in C++ using the
[bamtools](https://github.com/pezmaster31/bamtools) and AMOS C++ APIs.
The fruits of my labour was the program [sam2amos](https://github.com/ctSkennerton/sam2amos),
which can take a BAM file and convert it into either an AMOS message
stream or an AMOS bank.  I was honestly really impressed with the
simplicity of both of these APIs, which made writing this program
relatively easy.

After making my AMOS bank file with sam2amos I ran the `goBambus2.py` wrapping script,
which unfortunately resulted in bambus segfaulting. Bugger!  I didn't
have it in me to debug the situation and figure out whether it was caused by a problem in
sam2amos (which it probably is) or something else.

Four down, two to go...
## Finally some results
### scarpa
I never personally ran *scarpa* (one of my colleagues did) but it shared some of the
annoyances that *mip* had.  *Scarpa* assumes that the reads are in
forward-reverse orientation so it required that both of the reads be
reverse complemented before use.  Like *mip*, the reads needed to be
mapped independently.  But unlike *mip*, *scarpa* gave some results!!
Of course the output was a simple fasta file of scaffolds without any
indication of what contigs were actually put together.  A little more
information would have been nice...

### sspace
*Sspace* is the best of the bunch when it comes to usability.  You can
specify what orientation the reads were in and there was only one manual
preprocessing step required; *sspace* does not accept compressed files so 
the reads had to be passed through `ungzip` before use.  *Sspace* does however have some annoying 
automatic preprocessing steps. First, the reads are copied into the
output directory (why? I don't know). Second, *sspace* only runs bowtie,
you cannot specify any other mapper, this isn't so much of a problem but
it annoyingly repeats the mapping even if the results from a previous
run exist in the output directory.  The output though is better than
*scarpa's* as it gives you a fasta file of scaffolds, an "evidence file" actually 
telling you how it put contigs together and optionally a graphviz file
so that you can look at the contig linkages.

## What I want from a scaffolder
1. The ability to take either a SAM or BAM file of read mappings; or
2. If a particular read mapper is really required, give me the ability 
   to set the read orientation and used commpressed input files
3. One that uses pipes and does not copy reads into an output
   directory. The less file I/O the better
4. No configureation files, I want to pipeline this thing, give me some
   command-line options
5. Tell me how the scaffolding was done with some sort of easy tabular
   file format.  Better yet make that output format [AGP](http://www.ncbi.nlm.nih.gov/projects/genome/assembly/agp/AGP_Specification.shtml),
   which is required by NCBI when uploading scaffolds.

## Epilogue
Oh yeah, *sspace* gave better results than *scarpa*


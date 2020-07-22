---
title: "Finding 16s/18s reads in metagenomes"
date: 2014-04-19
categories: ['metagenomics', 'bioinformatics', 'tutorial']
---
Got a Metagenome? want to know what the community looks like?
<!-- more -->
rRNA operons are typically poorly assembled in metagenomic datasets due to
highly conserved sequences.  More targeted assembly approaches may be necessary
to obtain accurate reconstructions from short read datasets. There are a few
ways in which we can extract reads originating from either 16S or 18S reads and
there are a number of programs ([SSU-ALIGN](http://selab.janelia.org/software/ssu-align/),
[rRNASelector](http://www.ncbi.nlm.nih.gov/pubmed/21887657),
[riboPicker](http://ribopicker.sourceforge.net/),
[SortMeRNA](http://bioinfo.lifl.fr/RNA/sortmerna/),
blast, [bowtie](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml) or [bwa](https://github.com/lh3/bwa)) to name a few.

There are a other ways of looking at the communty from raw metagenomic
reads. These are mostly kmer-based approaches like
[kraken](http://ccb.jhu.edu/software/kraken/) or extended marker gene
approaches like
[MetaPhlAn](http://huttenhower.sph.harvard.edu/metaphlan) or
[phylosift](http://phylosift.wordpress.com/). I'm not against any of
them, just never used them, so I'm going to keep the following post
specific to 16s/18s community composition

## Extracting 16S/18S reads
I've used SortMeRNA and bowtie/bwa in my workflows with good success. The
difference between these two methods is that SortMeRNA that uses a kmer
searching method using an index created from a database of previously sequenced
genes whereas bowtie/bwa use a local alignment method to compare the query sequence to
a previously made database. Below are instructions on how to use both methods.

## Installing and using SortMeRNA
From the source page for SortMeRNA on [github](https://github.com/biocore/sortmerna)
copy the ssh clone url and then open up a terminal window and type
```bash
$ git clone git@github.com:biocore/sortmerna.git
$ cd sortmerna
$ ./configure --prefix=$PWD
$ make install
```
**Mac OSX install note:** I found that there was an error with my install
with `configure` complaining that it was missing `install-sh`. I solved this by
copying [autogen.sh](http://sourceforge.net/projects/buildconf/) into the same
directory as `configure`, execute it and then try running configure again.

The commands above should install SortMeRNA in the directory which you downloaded
it to.
#### Building indexes for SortMeRNA
SortMeRNA comes packaged with 8 different rRNA databases, all derived from SILVA.
Build either the 16S or 18S database, depending on what you want. Below is the
command used to generate the 18S index (assuming that you've installed in the
download directory)

```bash
$ bin/indexdb_rna --ref rRNA_databases/silva-euk-18s-database-id95.fasta,index/silva-euk-18s-database-id95 --sensitive
```

If you `ls` the index directory you should see four files generated.
#### Preprocessing reads
Now we can use this index to extract the reads that may come from 18S using the
`sortmerna` command. I'm going to assume that there are files containing raw
reads from an Illumina run called `file_R1.fastq.gz` and `file_R2.fastq.gz`.
Unfortunately one of the limitations of SortMeRNA is that it requires that you
only give it a single file and that the file is uncompressed. So to start with,
unzip the files using `gzip` and then combine them into a single file using
`merge-paired-reads.sh` found in the scripts directory of the SortMeRNA source
code

```bash
$ gunzip file_R1.fastq.gz file_R2.fastq.gz
$ bash scripts/merge-paired-reads.sh file_R1.fastq file_R2.fastq combined.fastq
```

To save space, it's probably best to re-zip the files to save on harddrive space

```bash
$ gzip file_R1.fastq file_R2.fastq
```

#### Extracting the reads
Now we can run `sortmerna`, saving matched reads (and their mates) to a file
with the prefix 'matched-18S' in fastq format using 4 processors.

```bash
$ bin/sortmerna --reads combined.fastq \
    --ref rRNA_databases/silva-euk-18s-database-id95.fasta,index/silva-euk-18s-database-id95 \
    --paired-in \
    --fastx \
    --aligned matched-18S \
    -a 4
```

## Installing and using BWA
My personal preference is to use bwa over bowtie but the merits of either are
debatable (there are also 50 other programs out there that try to solve the same
problem so the choice is yours). Download bwa from [github](https://github.com/lh3/bwa)
by copying the ssh clone url and typing the following into the terminal. (Change
directories out of the SortMeRNA source directory before you do this)

```bash
$ git clone git@github.com:lh3/bwa.git
$ cd bwa
$ make
```

bwa also requires that an index of the database be made. For simplicity, lets
use the same database that was included in SortMeRNA. To make the bwa index
copy the files that you want from the SortMeRNA source directory into a new
location.

```bash
$ cp ../sortmerna/rRNA_databases/silva-euk-18s-database-id95.fasta .
$ bwa index silva-euk-18s-database-id95.fasta
```

This should make a whole bunch of files that have extra file extensions appended
to the fasta file. With the index created we can now align the reads

```bash
$ bwa mem silva-euk-18s-database-id95.fasta file_R1.fastq.gz file_R2.fastq.gz > aligned_18S.sam
```
(Notice that there is no preprocessing steps necessary for the reads)
#### Postprocessing the sam file
The output of bwa is a standardized format called [sam](http://samtools.github.io/hts-specs/SAMv1.pdf)
which many programs will now output. This format essentially describes the
alignment of each of the query sequences to the reference sequences. This is
not exactly what we want, which is the reads that were successful hits to the
reference in fasta/q format. To go from a sam file to a fasta/q file is a little
complicated (I wish it wasn't). To start with, download [samtools](https://github.com/samtools/samtools)
and [fxtract](https://github.com/ctSkennerton/fxtract) from github, and download
them into new source directories (bonus points for getting them installed without
a walkthrough). First convert the sam file into its equivalent binary format and
filter out unaligned sequences:

```bash
$ samtools view -SubF 4 aligned_18S.sam | samtools sort - aligned_18S && samtools index aligned_18S.bam
```

Now get the names of all the reads that aligned:

```bash
$ samtools view aligned_18S.bam | cut -f 1 >aligned_reads.txt
```

And finally extract those reads from the original fastq files:

```bash
$ fxtract -Hf aligned_reads.txt file_R1.fastq.gz file_R2.fastq.gz >aligned_reads.fastq
```

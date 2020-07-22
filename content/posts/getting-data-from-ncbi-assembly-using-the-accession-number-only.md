---
title: 'Getting data from NCBI assembly using the accession number only'
date: Tue, 10 Apr 2018 23:18:11 +0000
draft: false
tags: ['Bioinformatics']
---

[NCBI's assembly database](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4702866/) is a great one-stop-shop for genomic data and annotations but it's actually kind of difficult to download data if you only know the accession number of an assembly. The documentation says that the assembly database is integrated with [entrez-direct](https://www.ncbi.nlm.nih.gov/books/NBK179288/), a great set of command line utilities for accessing NCBI data from the command line. Most of the databases have an option to download data based on the ID, so I thought that something like the following would work

```sh
efetch -id GCA_000398025.1 -db assembly -format fasta 
```

Alas it does not, however the directory structure for the assembly FTP site nicely mirrors the accession numbers so for the example above, the path looks something like this: ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/398/025/ Below is a little awk script that I wrote to generate the FTP path and for input into wget. The input for this script is a file containing the accession in the first column and optionally a second column containing the assembly name. The full FTP path contains both of these but we can get around not having the assembly name by using wildcards. Nope, you need both the accession and the assembly name for this snippet to work. But I wrote a more expansive [script in python that do it with just the accession number](https://github.com/ctSkennerton/scriptShed/blob/master/download_ncbi_assembly.py), you should probably use it instead.

```
BEGIN{OFS="/"}
{
  accession = $1;
  joined_acc = accession
  if (NF == 2) {
    joined_acc = joined_acc"_"$2;
  } else {
    joined_acc = joined_acc"*";
  }
  split($1, a, "_");
  sub(/\\.[[:digit:]]$/, "", a[2]);
  gsub(/.{3}/,"&/", a[2]);
  print accession"\tftp://ftp.ncbi.nlm.nih.gov/genomes/all",a[1],a[2],joined_acc,joined_acc"_genomic.fna.gz"
}
```
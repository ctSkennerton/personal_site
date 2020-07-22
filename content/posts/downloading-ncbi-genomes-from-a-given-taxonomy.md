---
title: 'Downloading NCBI genomes from a given taxonomy'
date: Sun, 20 Nov 2016 05:18:43 +0000
draft: false
tags: ['Bioinformatics']
---

The [Entrez Direct](https://www.ncbi.nlm.nih.gov/books/NBK179288/) toolkit is great for programmatic access to all of NCBI's resources. This little snippet below finds all of the refseq representative genomes for a given NCBI taxonomy ID, makes a little summary of the genomes downloaded and uses `wget` to download the genbank files from the Assembly FTP. Change the inital query in the first call to `esearch` to change what genomes are downloaded.

<!--more-->

```sh
esearch -db assembly -query "txid28890[Organism:exp] AND representative [PROP]" |
 efetch -format docsum |
 xtract -pattern DocumentSummary -element 
    AssemblyAccession SpeciesTaxid SpeciesName FtpPath_RefSeq |
 sed 's/,.\*//' |
 sort -k 3,3 |
 tee downloaded_genomes.tsv |
 cut -f 4 |
 sed -e 's/$/\\/\*genomic.gbff.gz/' |
 wget -i /dev/stdin
```
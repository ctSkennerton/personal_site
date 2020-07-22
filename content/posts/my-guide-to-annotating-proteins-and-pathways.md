---
title: 'My guide to annotating proteins and pathways'
date: Thu, 22 Sep 2016 22:14:50 +0000
draft: false
tags: ['Bioinformatics']
---

So. You've got yourself a nice new genome sequence and you want to know what kind of metabolism it has. There is a good chance that you have some idea already — you think it's a nitrogen fixer or a sulfate reducer etc. — based on other analyses and now it's time to strengthen your paper with a bit of genomic evidence.

Getting an initial annotation
-----------------------------

The vast majority of the genes in the genome are going to be hypothetical proteins; of the rest, a sizable chunk are going to be genes with a general sort of annotation like "ABC transporter" (which says nothing about what it's transporting), and the rest are going to be metabolic genes that you probably care about. The first thing that you want to do is to throw your genome into one of the automatic annotation pipelines ([prokka](https://github.com/tseemann/prokka), [rast](http://rast.nmpdr.org/), [img](https://img.jgi.doe.gov/), [ncbi](https://www.ncbi.nlm.nih.gov/genome/annotation_prok/), [kaas](http://www.genome.jp/tools/kaas/)) to get an initial annotation. You can then search through these to look for pathways of interest. However, don't just straight out believe what these pipelines say, they will definitely miss some genes and report annotations that aren't correct, but they are a good start since in most cases you will only be interested in a portion of the genes. The most important thing to remember is to be a scientist. Have a sceptical mindset and interrogate your data. You can't look at every gene (well you can but probably don't have time) so my rule of thumb is that you should manually check all of the genes in all of the pathways that you are going to write about in your paper.

My Workflow
-----------

### 1. Compare the sequence of your unknown gene against Uniprot. 

You can also use NCBI blast database or IMG or any other database you wish however I like Uniprot because it ranks it annotations with an [overall confidence score](http://www.uniprot.org/help/annotation_score), so you know what kind of experiments has informed the annotation. Uniprot will show some starred proteins that are biochemically verified - this is a good way to figure out if your gene is annotated with the correct function.

Remember that most genomes are automatically annotated which means that even if all of the top blast hits are annotated as the same thing, it doesn't mean that it's correct. Comparing it against genes that have been biochemically annotated is important.

Also remember that blast is a local aligner, so don't just look at the e-value, look at the percentage matched of the query (your gene) and the subject (gene in the database). If you have a very long gene and only part of it matches then you should definitely investigate this!

### 2. Find conserved domains using Interpro

Sometimes (read: Often) blast matches will not be conclusive or return only a general annotation. In these situations it's best to look at the protein sequence another way. In this case looking at the domains that make up the protein using Interpro may help figure out what it does. This is also a good way to compare your protein to a biochemically analysed protein. I your gene has extra domains, different domains, or fewer domains then it may well be that the blast annotation was incorrect.

### 3. Look at surrounding genes

A lot of enzymes are encoded by multiple genes which all exist in one operon. If you want to say that your bug has some enzyme you should make sure that it contains all of the subunits for that enzyme. This can be an essential step where enzymes evolved from a common ancestor such that individual subunits can be difficult to distinguish. [A good example here is NADH dehydrogenase and the evolutionarily related hydrogenases](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3144371/), which can be easily distinguished by the operon structure, but individual subunits often get mis-annotated.

### 4. Make an alignment (optional)

The last thing you might need to do is to make an alignment to see if your putative enzyme has all (or some of) of the right conserved residues as other biochemically characterised proteins. If the alignment looks terrible then perhaps you've got something novel.
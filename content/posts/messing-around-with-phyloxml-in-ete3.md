---
title: 'Messing around with Phyloxml in ete3'
date: Fri, 12 Aug 2016 22:09:04 +0000
draft: false
tags: ['Bioinformatics']
---

[ete3](http://etetoolkit.org/) has support for [phyloxml](http://www.phyloxml.org/) which I use with [archaeopteryx](https://sites.google.com/site/cmzmasek/home/software/archaeopteryx) tree viewer for a lot of my day-to-day phylogenetics visualisation. My main reason for using phyloxml is one of convenience as I have a script that will easily add in the proper organism name onto the tree and I think that archaeopteryx is a really good basic tree viewer. I wanted to draw a tree from phyloxml in ete using [my own style](https://connorskennerton.wordpress.com/2016/07/09/drawing-phylogenetic-trees-connor-style/) and to have the proper organism name to be rendered. In my phyloxml file I have this coded in as the scientific name for each leaf (see below for phyloxml snippet), so now all I needed to do was make this the node name when rendering the tree. 

```xml
<clade> 
  <name>IMG_2526164742</name> 
  <branch_length>0.19955</branch_length> 
  <taxonomy> 
    <scientific_name>Desulfobacterium anilini DSM 4660</scientific_name> 
  </taxonomy>
</clade> 
 ```
  Easy, right? Wrong. I found that the interface for phyloxml was not the same as for newick formatted trees and unfortunately the [documentation for phyloxml in ete3 is a bit lacking](http://etetoolkit.org/docs/latest/reference/reference_phyloxml.html) as there wasn't a complete listing of methods for each class. After much messing around, looking at the source code of ete3 and examining python objects using the builtin [`dir`](https://docs.python.org/3/library/functions.html#dir) function I was able to get what I wanted. turns out that for each node/leaf I needed to access the `phyloxml_clade` attribute, which has an attribute taxonomy, which implements an iterable interface (I think it's probably a list), which I could then use to access the scientific name and make the name of the leaf for printing. It's a little convoluted but easy when you know how. 
  
```python
from ete3 import Phyloxml
project = Phyloxml()

# iterate through the trees in the phyloxml file
for tree in project.get_phylogeny():
    # go through the node in the tree
    for node in tree:
        # assign the node name from the data in the phyloxml file
        node.name = node.phyloxml_clade.taxonomy[0].get_scientific_name()

tree.show()
```
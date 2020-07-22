---
title: 'An update on Bioawk'
date: Tue, 01 Oct 2019 17:26:23 +0000
draft: false
tags: ['Bioinformatics']
---

[Bioawk](https://github.com/lh3/bioawk) is a great project started by Heng Li some years ago. The aim was to take the [awk source code](https://github.com/onetrueawk/awk) and modify it slightly for use with common biological formats and adding in some new functions. Heng's original doesn't accept too many pull requests so to add in some features, I maintain [my own fork](https://github.com/ctSkennerton/bioawk) that has a few improvements.

Long ago I added in a translate function and recently I added in a function that will take the attribute string from a GFF file and turn it into an awk array. It was possible to split the GFF attribute string in awk itself as the following awk code demonstrates

```
function gffattr(attr, array){
    # split the attribute string into key value pairs
    split(attr, a, ";")
    for (i in a) {
        # now split out the key and value
        split(a[i], b, "=")
        for (j in b) {
            array[j] = b[j]
        }
    }
}
```

but this is a bit clunky and not suited for the kind of one-liner processing that awk excels at. So now with the latest update of my bioawk fork this function is built-in. If you want to filter a GFF on some attribute you can now do something like the following

```
bioawk -c gff '{gffattr($attribute, a); if(a["some_attr"] == "some_value") print}'
```

Other random updates
--------------------

The base awk source code has been getting a lot more love recently and my fork has so far merged in all of the upstream updates so there are a few new bug fixes in there.

Bioawk used to ignore the output field separator when printing the whole record -- this is no longer the case in my fork.
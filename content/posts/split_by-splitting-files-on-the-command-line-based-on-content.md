---
title: 'split_by: splitting files on the command line based on content'
date: Sun, 09 Apr 2017 08:04:41 +0000
draft: false
tags: ['Bioinformatics']
---

Unix has so many great ways to perform text manipulation but one niche which hasn't been filled is splitting a tabular file into pieces based on the contents of certain columns. There are two commands, `split` and `csplit`, that do a similar role. `split` can split a file into a certain number of bytes or lines; `csplit` uses a regular expression to determine where to split the file. Often for my purposes neither of these tools is a good fit, and what I really want is an equivalent to the "group by" clause in SQL databases. Group by sorts rows based on certain grouping columns so that you can then perform summaries on that group. `split_by` splits out a file based on the grouping columns for further processing on each chunk. For example, say I've got a delimited text file containing a mix of categorical and numerical data like the following:

|A|B|C|D|E|F|
|--- |--- |--- |--- |--- |--- |
|AU|BR|447|223.2|55958|US|
|AU|FR|348|16.8|58484|AU|
|US|UK|12|53.30|129|PG|

I want to split this file into smaller files based on what's in column A, which in this example would make two files for the 2 lines containing "AU" and for the 1 line containing "US". Neither, `split` or `csplit` really handle this case and the only option is to use an `awk`, `perl` or other script to handle it. (It's also possible to do this in multiple passes with `grep`) After writing similar awk one-liners to do this sort of thing I decided to make it a bit more reusable: 

```
#!/usr/bin/awk -f
BEGIN{
    split(cols, group_by, ",")
    col_sep = "_"
}
{
    if(header == 1 && NR == 1) {
        next
    }
    out_file = ""
    for(col in group_by) {
        if(col == 1) {
            out_file = group_by[col]
        } else {
            out_file = out_file col_sep group_by[col]
        }
    }
    out_file = (prefix out_file suffix)
    print $0 > out_file
}
```

 That's the whole program! You can run it like the following, taking advantage of the variables defined in the script:
 
 ```bash
./split_by -vFS="\t" -vprefix=test_split_ -vsuffix=".tsv" -vcols=1 -vheader=1 file.tsv
```

This will produce two files on the example above called "test\_split\_AU.tsv" and "test\_split\_US.tsv". Only the _cols_ argument is needed to say which of the columns is used to split on. If you want to split on multiple columns pass a comma separated list `-vcols=1,2`. The output file names are generated based on the contents of those columns so if there are any special characters in there that might mess up a file name, you're in trouble. The _prefix_ and _suffix_ variables default to nothing but are useful for being able to find the files for later by giving them all a common prefix. The nice thing about this is that all of the `awk` variables are still available so changing between a csv and tsv file can be achieved using the _FS_ parameter on the command line.
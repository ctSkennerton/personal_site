---
title: 'Formatting scientific names in R'
date: Sat, 22 Jul 2017 17:27:02 +0000
draft: false
tags: ['Bioinformatics', 'R']
---

Here is a function that will take a character string in R and return an expression for fancy formatting in plots that properly italicize scientific names. The syntax for doing this is truly quite horrible, but this is how R does it. 

<!--more-->

```R
scientific_name_formatter <- function(raw_name) { 
    # strsplit returns a list but we are passing in only 
    # one name so we just take the first element of the list 
    words = strsplit(raw_name, ' ', fixed = TRUE)[[1]] 
    # some sort of acronym or bin name, leave it alone 
    if(length(words) < 2) { return(raw_name) } 
    else if (length(words) > 2) { 
        if (words[1] == 'Candidatus') { 
            # for candidatus names, only the Candidatus part is italicised # name shortening it for brevity 
            unitalic <- paste(words[2:length(words)], collapse = ' ') return(bquote(paste(italic(Ca.) ~ .(unitalic)))) } 
        else if (grepl('^\[A-Z\]+$', words[1])) { 
            # If the first word is in all caps then it is an abreviation 
            # so we don't want to italicize that at all 
            return(raw_name) } 
        else { 
            # assume that everything after the second word is strain name # which should not get italicised 
            unitalic <- paste(words[3:length(words)], collapse = ' ') return(bquote(paste(italic(.(words[1])) ~ italic(.(words[2])) ~ .(unitalic) ))) 
        } 
    } else { 
        return(bquote(paste(italic(.(words[1])) ~ italic(.(words[2]))))) 
    } 
}
```
 I use it like the following in ggplot by setting the labels in `scale_y_discrete`. 
 
 ```R
 scale_y_discrete(limits = df$organism_name, labels = sapply(df$organism_name, scientific_name_formatter ))
 ```
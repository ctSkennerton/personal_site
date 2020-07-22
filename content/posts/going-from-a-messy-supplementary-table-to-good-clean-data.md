---
title: 'Going from a messy supplementary table to good clean data'
date: Mon, 11 Jul 2016 21:00:53 +0000
draft: false
tags: ['Bioinformatics', 'R', 'tidy data']
---

> Bioinformatics... Or 'advanced file copying' as I like to call it.
> 
> — Nick Loman (@pathogenomenick) [January 29, 2014](https://twitter.com/pathogenomenick/status/428616138252353536)

  Get ready for some advanced file copying! I recently had to clean up some data from the supplementary material from [Pereira _et. al_ 2011](http://journal.frontiersin.org/article/10.3389/fmicb.2011.00069/abstract), which is a very nice table of manually annotated genes in sulfate reducing bacteria. The only problem is that the table is designed for maximum human readability, which made it a real pain when trying to parse out the data. I decided to use `R` and the “[tidyverse](https://github.com/tidyverse)” packages to clean up the table to make things work better for downstream analyses. This isn’t part of my normal workflow, I’m more of a python guy, but after doing this analysis in `R` I’d have to say that I’m a convert.

#### Getting the data

Download the supplementary material from the paper linked above, if you want to play along at home. This data is a nicely formatted excel workbook containing eight spreadsheets with the locus identifiers for a number of genes important in suflate reducing prokaryotes. While this data is nice and visually appealing, it is not [tidy](https://cran.r-project.org/web/packages/tidyr/vignettes/tidy-data.html) and it’s difficult to get the information I want out of it. I want the locus identifiers for the genes of interest so that I can download them from NCBI and use them as a blast database.

#### Cleaning the data

Some things are just easier to do in excel before tidying the data in `R` here is what I did:

1.  removed the empty columns and rows at the beginning. This actually isn’t difficult to do in `R`, but doing this makes inputting the data more painless cause then `R` will pick up the column names.
2.  Remove the rows that contain just the taxonomic information
3.  For some of the sheets (‘Hases’, for example) I removed rows at the beginning that gave hierarchy to the columns. These are mostly unnecessary and make it difficult to parse the excel sheet, as `readxl` does not currently handle merged cells and cause the boundaries of this hierarchy is coded visually using cell boarders in excel.
4.  For some reason there were single quotation marks in the _Archaeoglobus fulgidus_ DsrK locus identifiers, which I removed

Open up an `R` session and load the following libraries (assuming that you already have them installed)

```R
library(tidyr)
library(readxl)
library(stringr)
library(dplyr)
```
Import the data into `R` using `readxl`. Creates a list of dataframes.

```R
d <- lapply(excel_sheets("~/Downloads/data_sheet_2.xls"), \
read_excel, path = "~/Downloads/data_sheet_2.xls")
```

remove the completely empty rows

```R
d <- lapply(d,
function(n) n[rowSums(is.na(n)) != ncol(n),])
```

Lets look at what our table looks like (Note the ‘organism’ column is not shown for brevity)

```R
knitr::kable(d[[1]][,-1])
```

|locus|SAT|AprA|QmoA|DsrA|DsrC|H-Ppi|FdxI|FdxII|
|--- |--- |--- |--- |--- |--- |--- |--- |--- |
|AF|1667|1670|0663|0423|2228|NA|00427; 1010; 0355; 0923; 2142; 0166; 1700; 0156; 0464|0167|
|Arcpr_|1264|1261|1260|0139|1726|NA|0142; 0625; 0483; 0712; 1058|NA|
|Cmaq_|0274|0273|NA|0853|0856|0949|0549; 1711|NA|
|DaesDRAFT_|2031|2029|2028|2438|0796|NA|1729|0903|
|Dde_|2265|1110|1111|0526|0762|NA|3775|0286|
|Ddes_|0454|2129|2127|2275|1917|NA|889|NA|
|DMR_|39470|05400|05410|03600|15890|NA|39570; 18760|13970|
|DESPIG_|02241|02773|02771|NA|02353|NA|00991|NA|
|Desal_|0228|0230|0231|0787|0984|NA|1299|0241; 2850|
|DFW101DRAFT_|0832|1162|1163|3451|2958|NA|0847|0729|
|DVU|1295|0847|0848|0402|2776|NA|3276|NA|
|Dbac_|3196|3198|3199|0279|2958|NA|0275|2977|
|Dalk_|2445|1569|1568|4301|4140|3373|4380; 2230; 2714|2374|
|HRM2_|31180|04510|04500|42400|22050|NA|26720; 10680; 01580; 39570|40690|
|Dole_|1002|0998|0999|2669|0463|2820|1168|2655|
|Dret_|1968|1966|1965|0244|1739|NA|0240|0154; 0169|
|DthioDRAFT_|1410|1407|1406|2272|2675|NA|2268|NA|
|DP|1472|1105|1106|0797|0997|NA|2755; 0775|1865|
|DaAHT2_|0293|1471|1470|0296|2041|NA|1668|2532; 2287|
|Sfum_|1046|1048|1287|4042|4045|3037|4046|2933; 3217|
|Dtox_|3579|3577|3576|0079|0077|3931|0074; 0532; 1221; 1608; 3208|1637|
|Dred_|0635|0637|0638|3187|3197|2985|3200; 2937; 0466|2203|
|Daud_|1076|NA|1884|2201|2190|0308|2193; 1963|1080|
|Adeg_|1712|1080|1079|2094|0035|NA|0032|1625|
|THEYE_|A1835|A1832|A1831|A1994|A0003|NA|A0789|NA|

In the data, the columns for each gene are really values, not variables; they should be converted into individual rows. To do this use the `gather` function from `tidyr`. Here I specify the name of the new columns `gene.identifier` which will contain the name of the gene and `locus.identifier` which will contain the information for that gene. I’m also setting `na.rm` which will not include genes where it was not found in the organism. After the gather function is applied all of the data frames in the list will have the same columns, which means that they can all be concatenated into one big data frame. To do this I’m using `dpylr::bind_rows`.

```R
d <- lapply(d, function(n){ n %>% 
gather(gene.identifier, locus.identifier, 
-c(organism, locus), na.rm=TRUE)}) 
d <- bind_rows(d)
## Warning in rbind_all(x, .id): Unequal factor levels: coercing to character
knitr::kable(d[130:140,])
```

|organism|locus|gene.identifier|locus.identifier|
|--- |--- |--- |--- |
|Archaeoglobus fulgidus|AF|FdxI|00427; 1010; 0355; 0923; 2142; 0166; 1700; 0156; 0464|
|Archaeoglobus profundus|Arcpr_|FdxI|0142; 0625; 0483; 0712; 1058|
|Caldivirga maquilingensis|Cmaq_|FdxI|0549; 1711|
|Desulfovibrio aespoeensis|DaesDRAFT_|FdxI|1729|
|Desulfovibrio desulfuricans G20|Dde_|FdxI|3775|
|Desulfovibrio desulfuricans ATCC 27774|Ddes_|FdxI|889|
|Desulfovibrio magneticus RS-1|DMR_|FdxI|39570; 18760|
|Desulfovibrio piger|DESPIG_|FdxI|00991|
|Desulfovibrio salexigens|Desal_|FdxI|1299|
|Desulfovibrio sp. FW1012B|DFW101DRAFT_|FdxI|0847|
|Desulfovibrio vulgaris Hildenborough|DVU|FdxI|3276|

The other untidy aspect of the dataset is that there are multiple locus identifiers for some of the genes (presumably cause there are multiple copies in the genome). We next need to split them out into separate observations (rows). The `str_split` function from `stringr` will split a string based on a regular expression and return a list of values. I then pass this to the `unnest` function, which will “flatten” the list into multiple rows as required.

```R
d %>% mutate(locus.identifier = 
str_split(as.character(locus.identifier), "; |\\/")) %>% 
unnest(locus.identifier) -> d
knitr::kable(d[130:140,])
```

|organism|locus|gene.identifier|locus.identifier|
|--- |--- |--- |--- |
|Archaeoglobus fulgidus|AF|FdxI|00427|
|Archaeoglobus fulgidus|AF|FdxI|1010|
|Archaeoglobus fulgidus|AF|FdxI|0355|
|Archaeoglobus fulgidus|AF|FdxI|0923|
|Archaeoglobus fulgidus|AF|FdxI|2142|
|Archaeoglobus fulgidus|AF|FdxI|0166|
|Archaeoglobus fulgidus|AF|FdxI|1700|
|Archaeoglobus fulgidus|AF|FdxI|0156|
|Archaeoglobus fulgidus|AF|FdxI|0464|
|Archaeoglobus profundus|Arcpr_|FdxI|0142|
|Archaeoglobus profundus|Arcpr_|FdxI|0625|

Now for the final clean-up. The original data separated the locus prefix and the locus identifier, now I want to combine them back together. To do this I’m going to use a couple of calls to the `mutate` function, which modifies a column. First, in the previous command I converted the `locus.identifier` column to characters, however this has the unwanted effect of having decimal places in the strings, which I don’t want. Passing the `locus.identifier` column to the `sub` function will remove the unwanted text. The next `mutate` command combines the `locus` and `locus.identifier` columns into one and finally I select which columns I want in the final data frame using the `select` function.

```R
d %>% mutate(locus.identifier = 
sub("\\.0+","",locus.identifier, perl=T)) %>% 
mutate(locus = paste0(locus,locus.identifier)) %>% 
select(organism, locus, gene.identifier) -> d
knitr::kable(d[1:10,])
```

|organism|locus|gene.identifier|
|--- |--- |--- |
|Archaeoglobus fulgidus|AF1667|SAT|
|Archaeoglobus profundus|Arcpr_1264|SAT|
|Caldivirga maquilingensis|Cmaq_0274|SAT|
|Desulfovibrio aespoeensis|DaesDRAFT_2031|SAT|
|Desulfovibrio desulfuricans G20|Dde_2265|SAT|
|Desulfovibrio desulfuricans ATCC 27774|Ddes_0454|SAT|
|Desulfovibrio magneticus RS-1|DMR_39470|SAT|
|Desulfovibrio piger|DESPIG_02241|SAT|
|Desulfovibrio salexigens|Desal_0228|SAT|
|Desulfovibrio sp. FW1012B|DFW101DRAFT_0832|SAT|
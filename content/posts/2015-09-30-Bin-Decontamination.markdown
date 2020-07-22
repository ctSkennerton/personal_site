---
title: "Genome Bin Decontamination"
date: "2015-10-03"
tags: ['bioinformatics']
---
Genome bins comming off automated pipelines can be contaminated
with parts of other genomes. As part of my workflow I use
[CheckM](http://genome.cshlp.org/content/early/2015/05/14/gr.186072.114.abstract)
(I'm biased since I'm a coauthor) to assess the contamination of
genome bins using single-copy marker genes. If you're lucky then
the genome bins that you're interested in will be relatively complete
without much contamination. Unfortunately that isn't always the
case. In this blog post I'm going to run through some of the analyses
that I did on a genome bin that was 90% complete but 70% contaminated.
This is exploratory analysis to see if I can manually improve the
bin over what the automated tools can do.

<!--more-->

## Background
I'm not going into details on how I got this bin but briefly I had
three metagenomic samples, named 3730, 5133, and 5579, which were
assembled with
[megahit](http://bioinformatics.oxfordjournals.org/content/31/10/1674.long)
and binned using [metabat](https://peerj.com/articles/1165/). I ran
the genome bins from sample 3730 through CheckM. In the below analysis I'm going to
try to improve **bin_41**.

## Analysis
### Creating the input files
For starters I want the `.depth.txt` file created by metabat, which I copied and renamed `3730_coverage.tsv`. Next I created a mapping file for which contigs belonged to which bins. I have the fasta files of all of the bins in a separate directory. To get the mapping of the fasta files I ran the following:
```sh
grep -oP '(?<=^\>)\S+' *fa | awk 'BEGIN{FS=":";OFS="\t"}{print $2,$1}' > 3730_bin_mapping.tsv
```
And finally I wanted to know where all of the multi-copy markers were so I created a file based on the CheckM output, with a bit of reformatting in awk:
```sh
checkm qa -o 6 ckm_results/lineage.ms ckm_results | awk 'BEGIN{FS="\t";OFS=","}{n = split($3,a,",");for(i = 1; i <= n; ++i){print $1,$2,a[i]}}' > 3730_multiple_markers.csv
```
### Exploratory analysis in R

```R
library(readr)
duplicated_markers = read_csv("3730_multiple_markers.csv")
coverage = read_tsv("3730_coverage.tsv")
# no header in the file, so give the columns names
bin_mapping = read_tsv("3730_bin_mapping.tsv", col_names = c("contigName", "bin"))
```
Clean up the dataframes to make the column names consistent and remove a bit of unneeded text. Note that these commands are specific to how files were named on my system, you may not need to do this or change this section to meet your own needs. 

```R
bin_mapping$bin = gsub("metabat_binned/final.contigs.fa.metabat_", "", bin_mapping$bin)
bin_mapping$bin = gsub("\\.fa$", "", bin_mapping$bin)
duplicated_markers$`Bin Id` = gsub("final.contigs.fa.metabat_", "", duplicated_markers$`Bin Id`)
duplicated_markers$`Gene Ids` = gsub("_\\d+$", "", duplicated_markers$`Gene Ids`, perl=TRUE)
colnames(duplicated_markers) <- c("bin", "marker", "contigName")
```
Create a smaller dataframe of just the binned contigs

```R
binned_contigs = merge(bin_mapping, coverage, by="contigName")
```
We are interested in Bin 41, lets get just the data for it.

```R
bin_41 = binned_contigs[binned_contigs$bin == "bins_41",]
```
Since we have three samples we can look at the coverage of the contigs in each sample as a matrix of 2D

```R
pairs(~final.contigs.3730.bam+final.contigs.5133.bam+final.contigs.5579.bam, data=bin_41, main = "Comparison of contig coverage between samples", labels = c("3730", "5133", "5579"))
```

![plot of chunk bin-decontamination-paired-coverage](/media/bin-decontamination-paired-coverage-1.svg) 
The majority of the contigs are found in sample 3730 between 10-15x and with very low coverage in sample 5133. Some of the contigs also have some coverage in sample 5579, but most don't. It's tempting to remove all of the contigs that don't fit into the band of coverage, but from this we can't be certain if the duplicated markers are in these outlier contigs.

Since the coverage of sample 5133 doesn't really factor into things lets look at a single 2D plot of the 3730 coverage and the 5579 coverage. As a coarse approach lets take a look at where all of the contigs with duplicated markers are in this plot. For starters make a new column in the bin_41 dataframe that tells us if the contig contains any duplicated marker.

```R
bin_41_duplicated_markers = subset(duplicated_markers, bin == "bins_41")
bin_41$containsDuplicateMarker <- bin_41$contigName %in% unique(bin_41_duplicated_markers$contigName)
```
And now lets plot the data, changing the color of the points based on the value of the 'containsDuplicateMarker' column

```R
plot(bin_41$final.contigs.3730.bam, bin_41$final.contigs.5579.bam, pch = 19, col = "lightgrey", cex=0.5, xlab="3730 coverage", ylab="5579 coverage", main="Contigs containing duplicated markers")
points(bin_41[bin_41$containsDuplicateMarker,]$final.contigs.3730.bam, bin_41[bin_41$containsDuplicateMarker,]$final.contigs.5579.bam, col = "red")
```

{{% figure src="/media/bin-decontamination-all-duplicate-markers-1.svg" %}}

Apart from a couple of outliers, the majority of the contigs that contain multiple markers are in the central mass of contigs. There doesn't appear to be any way to systematically remove a substantial amount of contamination in this genome bin.

Instead of looking at all contigs that contain any duplicated markers, we can also visualise the positions of contigs for a specific duplicated marker. You can get a list of the markers that are duplicated in the bin by using the unique command. Then create a logical vector of bin_41 of the contigs that contain any of the particular markers.


```R
unique(bin_41_duplicated_markers$marker)

##   [1] "PF03439.8"  "PF01172.13" "PF00164.20" "PF09173.6"  "PF03874.11"
##   [6] "PF00203.16" "PF00466.15" "PF00416.17" "PF05670.8"  "PF04127.10"
##  [11] "PF01090.14" "PF01193.19" "PF09249.6"  "PF11987.3"  "PF00298.14"
##  [16] "PF00252.13" "PF01198.14" "PF01201.17" "PF01351.13" "PF04567.12"
##  [21] "PF01922.12" "PF01667.12" "PF00398.15" "TIGR03685"  "PF04983.13"
##  [26] "TIGR03683"  "PF04561.9"  "PF04565.11" "PF04919.7"  "TIGR00425" 
##  [31] "PF01200.13" "PF07541.7"  "PF00679.19" "PF01194.12" "PF13656.1" 
##  [36] "PF01015.13" "PF01282.14" "PF01287.15" "PF00276.15" "PF01780.14"
##  [41] "TIGR00064"  "TIGR00389"  "PF04997.7"  "PF04563.10" "PF03947.13"
##  [46] "PF01912.13" "PF00181.18" "PF04010.8"  "TIGR00549"  "TIGR01018" 
##  [51] "PF01157.13" "TIGR00392"  "PF00347.18" "PF00831.18" "TIGR01080" 
##  [56] "TIGR00408"  "TIGR00270"  "PF08068.7"  "PF00687.16" "PF04560.15"
##  [61] "PF05000.12" "PF00380.14" "PF00752.12" "PF09377.5"  "TIGR02389" 
##  [66] "TIGR00419"  "PF01000.21" "PF04566.8"  "PF00562.23" "PF00366.15"
##  [71] "TIGR00057"  "PF01191.14" "PF00572.13" "PF01981.11" "PF01984.15"
##  [76] "TIGR03677"  "PF00411.14" "PF01864.12" "PF00238.14" "TIGR03724" 
##  [81] "PF00177.16" "PF00827.12" "TIGR02338"  "PF01849.13" "TIGR00468" 
##  [86] "PF01866.12" "PF01982.11" "PF00237.14" "PF03876.12" "PF02978.14"
##  [91] "TIGR00670"  "PF00623.15" "PF01246.15" "PF00573.17" "PF00189.15"
##  [96] "TIGR01213"  "TIGR00344"  "PF01868.11" "PF03764.13" "PF00867.13"

contains_x_marker = bin_41$contigName %in% subset(bin_41_duplicated_markers, marker == "PF01157.13")$contigName
```
With this vector we can plot the points like we did with the complete set of duplicated markers.

```R
plot(bin_41$final.contigs.3730.bam, bin_41$final.contigs.5579.bam, pch = 19, col = "lightgrey", cex=0.5, main="Position of contigs with PF01157.13", xlab="3730 coverage", ylab="5579 coverage")
points(bin_41[contains_x_marker,]$final.contigs.3730.bam, bin_41[contains_x_marker,]$final.contigs.5579.bam, col = "red")
```

{{% figure src="/media/bin-decontamination-single-marker-positions-1.svg" %}}

So now we can see that ther are three contigs containing this marker, one
appears to be an outlier, but the other two contigs have quite similar
coverage profiles

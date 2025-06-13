---
title: Common Bioinformatics Tasks With UNIX Commands
date: 2016-10-20T00:00:00+08:00
tags: ['bioinformatics', 'unix', 'text manipulation']
type: post
---

### awk

__1. Give names to each peak in a 3-column bed file:__

```bash
$ cat Oct4_union_peaks.bed

chr1 3062833 3063056
chr1 3343519 3343843
chr1 3482630 3483324
chr1 3549654 3549895
chr1 3648929 3649295

$ awk 'BEGIN{OFS="\t"}{print $1,$2,$3,"Oct4_union_peak_" NR}' Oct4_union_peaks.bed

chr1 3062833 3063056 Oct4_union_peak_1
chr1 3343519 3343843 Oct4_union_peak_2
chr1 3482630 3483324 Oct4_union_peak_3
chr1 3549654 3549895 Oct4_union_peak_4
chr1 3648929 3649295 Oct4_union_peak_5
```

__2. Print the number of fileds for each line:__

```bash
awk '{print NF}' some_file.txt
```

__3. Print the last column of a text file:__

```bash
awk '{print $NF}' some_file.txt
```

__4. In a [narrowPeak format](https://genome.ucsc.edu/FAQ/FAQformat#format12) file, Get all peaks that have more than 5 fold enrichment:__

```bash
awk '$7>5' Oct4_peaks.narrowPeak
```

__5. In a tab-delimited file, output the lines where the 3rd column is less than the 2nd column:__

```bash
# in a normal bed file, this should output nothing as the 3rd column is alway larger than the 2nd column
awk '$3<$2' Oct4_peaks.bed
```

__6. Only output the read names, the DNA sequences or the quality strings of a fastq file:__

```bash
# output read names
awk 'NR%4==1' Oct4_ChIP.fastq
 
# output DNA sequence
awk 'NR%4==2' Oct4_ChIP.fastq
 
# output quality strings
awk 'NR%4==0' Oct4_ChIP.fastq
```

__7. Add a “chr” at the beginning of a bed file coordinates (useful to convert non mitochondrial Ensembl coordinates to UCSC coordinates):__

```bash
awk '{print "chr" $0}' Oct4_peaks_ensembl.bed
```

__8. Calculate the length of each region in a bed file and add the length to the end of each region:__

```bash
awk 'BEGIN{OFS="\t"}{print $0, $3-$2}' Oct4_peaks.bed
```

__9. Get the summit point (highest pileup) from a narrowPeak file and output as a 4-column bed file:__

```bash
# simply output the summit point
awk 'BEGIN{OFS="\t"}{print $1,$2+$10,$2+$10+1,$4}' Oct4_peaks.narrowPeak
 
# output the summit point sorted by fold enrichment (highest to lowest)
sort -k7,7nr Oct4_peaks.narrowPeak | awk 'BEGIN{OFS="\t"}{print $1,$2+$10,$2+$10+1,$4}'
```

### grep

__1. Output the record names from a fasta file:__

```bash
grep '>' your_fasta.fa
```

__2. Display one line before the match and two lines after the match (-A and -B options):__

```bash
grep 'AAGCAGTGGTATCAACGCAGAGTACAT' -A 2 -B 1 input.fastq
```

__3. Only display the matching strings, not the entire line (-o option):__

```bash
# search for the P7 end of adapter sequence of a Nextera library
grep -o -P 'CCGAGCCCACGAGAC[ACGT]{8}ATCTCGTATGCCGTCTTCTGCTTG' *.fastq

# if you want to suppress output file name, use '-h' as well
grep -oh -P 'CCGAGCCCACGAGAC[ACGT]{8}ATCTCGTATGCCGTCTTCTGCTTG' *.fastq
```

### cut

__1. Extract the 1st, 2nd, 4-8th, 10th and following columns from a tab-delimited file (default):__

```bash
cut -f 1,2,4-8,10- input.txt
```

__2. Extract the 4-8th columns of a comma-delimited file:__

```bash
cut -f 4-8 -d, input.txt
```

__3. Extract the last column and use '/' as a delimiter:__

```bash
cat input.txt | rev | cut -f 1 -d/ | rev
```

### xargs

__1. Kill all pending jobs in LSF:__

```bash
bjobs | grep 'PEND' | awk '{print $1}' | xargs bkill
```

__2. Remove all .sh files that have the word 'foo' in them:__

```bash
grep -w 'foo' *.sh | cut -f 1 -d ':' | sort -u | xargs rm
```
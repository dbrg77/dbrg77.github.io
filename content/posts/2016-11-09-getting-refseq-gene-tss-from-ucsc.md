---
title: Getting RefSeq gene TSS from UCSC
date: 2016-11-09T00:00:00+08:00
tags: ['bioinformatics', 'unix', 'text manipulation', 'UCSC genome browser']
type: post
---

As a practice of using the unix command lines, let's get the transcription start site (TSS) coordinates of all hg38 RefSeq genes from the UCSC genome browser, and change it to a proper bed format:

__1. Go to the [UCSC genome browser](http://genome-euro.ucsc.edu/index.html).__

__2. Go to the table browser: Tools –> Table Browser.__

__3. Choose the following options:__

> __clade:__ Mammal  
> __genome:__ Human  
> __assembly:__ Dec. 2013 (GRCh38/hg38)  
> __group:__ Genes and Genes Predictions  
> __track:__ RefSeq Genes  
> __table:__ refGene  
> __region:__ genome  
> __output:__ all fields from selected table  
> __output file:__ whatever name you want (e.g. refseq_genes_hg38.txt)

__4. The downloaded text file is a tab-delimited file. You can get a rough idea what it contains:__

```bash
$ head -2 refseq_genes_hg38.txt

#bin name chrom strand txStart txEnd cdsStart cdsEnd exonCount exonStarts exonEnds score name2 cdsStartStat cdsEndStat exonFrames
0 NM_001276351 chr1 - 67092175 67134971 67093004 67127240 8 67092175,67095234,67096251,67115351,67125751,67127165,67131141,67134929, 67093604,67095421,67096321,67115464,67125909,67127257,67131227,67134971, 0 C1orf141 cmpl cmpl 0,2,1,2,0,0,-1,-1,
```

__5. txStart and txEnd indicate where the transcript starts or ends respectively. If it is from the + strand, then txStart is the TSS; if it is from the – strand, then txEnd is the TSS. Remember in a bed format, the start coordinates are 0-based, and the end coordinates are 1-based. Therefore, write a script called `getTSS.sh` and the content look like this:__

```bash
tail -n +2 refseq_genes_hg38.txt | awk '
BEGIN{OFS="\t"}{
if($4=="+") {print $3,$5,$5+1,$2 "_" $13,".",$4}
 else {print $3,$6-1,$6,$2 "_" $13,".",$4}
 }' > refseq_hg38_UCSC_TSS.bed
 
# Column 2 (name, $2) is the unique refseq ID
# Column 13 (name2, $13) is the gene name
# We want both of them for easy reading,
# that's why I put $2 "_" $13 there
```

__6. Now have a look the output:__

```bash
$ head refseq_hg38_UCSC_TSS.bed
chr1 67134970 67134971 NM_001276351_C1orf141 . -
chr1 67134970 67134971 NM_001276352_C1orf141 . -
chr1 67134970 67134971 NR_075077_C1orf141 . -
chr1 201283450 201283451 NM_000299_PKP1 . +
chr1 201283450 201283451 NM_001005337_PKP1 . +
chr1 8423686 8423687 NM_001042682_RERE . -
chr1 8817639 8817640 NM_001042681_RERE . -
chr1 8817639 8817640 NM_012102_RERE . -
chr1 34165841 34165842 NM_052896_CSMD2 . -
chr1 34165273 34165274 NM_001281956_CSMD2 . -
```

__7. Sometimes, different isoforms have the same TSS. To get the unique one, do this:__

```bash
sort -k1,1 -k2,2n -k3,3n -k6,6 -u refseq_hg38_UCSC_TSS.bed > refseq_hg38_UCSC_TSS_unique.bed
```

__8. Now have a look at the final output:__

```bash
$ head refseq_hg38_UCSC_TSS_unique.bed
chr1 11872 11873 NR_046018_DDX11L1 . +
chr1 17435 17436 NR_128720_MIR6859-4 . -
chr1 29369 29370 NR_024540_WASH7P . -
chr1 30364 30365 NR_036268_MIR1302-11 . +
chr1 36080 36081 NR_026818_FAM138A . -
chr1 69089 69090 NM_001005484_OR4F5 . +
chr1 140565 140566 NR_039983_LOC729737 . -
chr1 187957 187958 NR_128720_MIR6859-4 . -
chr1 206596 206597 NR_026823_FAM138D . -
chr1 451677 451678 NM_001005221_OR4F29 . -
```

__Note: in the original post, there was a mistake in the `getTSS.sh`. The `$5-1, $5` should be changed to `$5, $5+1`. This has been corrected now. Thank Rohit Satyam for pointing that out.__

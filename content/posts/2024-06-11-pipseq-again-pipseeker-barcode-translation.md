---
title: "PIPseq again - PIPseeker$^{\\textmd{TM}}$ Barcode Translation"
date: 2024-06-11T09:00:00+08:00
tags: ['single cell genomics', 'sequencing library', 'illumina sequencing', 'fluentbio', 'pipseq', 'pipseeker']
draft: false
showTableOfContents: true
type: post
---

In [**one of the post**](/posts/2023-06-03-guessing-library-structures-2/) where we figured out the library structure of the PIPseq$^{\textmd{TM}}$ V3 chemistry last year, we left a cliffhanger about how the [PIPseeker$^{\textmd{TM}}$](https://www.fluentbio.com/products/pipseeker-software-for-data-analysis/) software from **FluentBio** performed the barcode "translation". During the translation, an original cell barcode that consists of four parts of "sub-barcodes" (8 bp + 6 bp + 6 bp + 8 bp) got mapped to a 16-bp barcode.

## The V3 data

Using the species mixing data from the V3 chemistry mentioned in [the previous post](/posts/2023-06-03-guessing-library-structures-2/), we would end up with the following files in our working directory if we finished all procedures described in the previous post:

```
fluentbio
├── barcode
│   ├── barcoded_fastqs
│   │   ├── barcoded_1_R1.fastq.gz
│   │   ├── barcoded_1_R2.fastq.gz
│   │   ├── barcoded_2_R1.fastq.gz
│   │   ├── barcoded_2_R2.fastq.gz
│   │   ├── barcoded_3_R1.fastq.gz
│   │   ├── barcoded_3_R2.fastq.gz
│   │   ├── barcoded_4_R1.fastq.gz
│   │   └── barcoded_4_R2.fastq.gz
│   ├── barcodes
│   │   ├── barcode_whitelist.txt
│   │   └── generated_barcode_read_info_table.csv
│   ├── logs
│   │   ├── pipseeker_2023-05-25_16-53-40.log
│   │   └── progress.log
│   ├── metrics
│   │   └── barcode_stats.json
│   └── run_config.json
├── fastqs
│   ├── RK20220802_FR_3_S3_L001_R1_001.fastq.gz
│   ├── RK20220802_FR_3_S3_L001_R2_001.fastq.gz
│   ├── RK20220802_FR_3_S3_L002_R1_001.fastq.gz
│   └── RK20220802_FR_3_S3_L002_R2_001.fastq.gz
├── fb_v3_bc1.tsv
├── fb_v3_bc2.tsv
├── fb_v3_bc3.tsv
└── fb_v3_bc4.tsv
```

Again, using the read `@VH00284:1:AAANWCCHV:1:1101:63733:1057` from the previous post as an example, we see that the read is like this in the original fastq file:

```bash
$ zcat fastqs/RK20220802_FR_3_S3_L001_R1_001.fastq.gz | grep -A 3 '@VH00284:1:AAANWCCHV:1:1101:63733:1057'
@VH00284:1:AAANWCCHV:1:1101:63733:1057 1:N:0:GGCCTCCT+AGAGGATA
TTGGGTCCATGCAACACGAGTTGGTATCGAGTGGGAAGTGCGGACCTGAGT
+
C;C!CCCCCCCCCCC-;CCCC;CCCCCCCCCCCCCCC!CCCCCCCCCCCCC 
```

The same read is like this in the formatted fastq file by PIPseeker$^{\textmd{TM}}$:

```bash
$ zcat barcode/barcoded_fastqs/barcoded_1_R1.fastq.gz | grep -A 3 '@VH00284:1:AAANWCCHV:1:1101:63733:1057'
@VH00284:1:AAANWCCHV:1:1101:63733:1057 1:N:0:GGCCTCCT+AGAGGATA
AGGCCAAAACCCCCAAGCGGACCTGAGT
+
~~~~~~~~~~~~~~~~CCCCCCCCCCCC
```

Since now we know the library structure of the V3 chemistry should be:

```
[8-bp bc1]ATG[6-bp bc2]GAG[6-bp bc3]TCGAG[8-bp bc4][12-bp UMI]
```

There are 96 unique sequences of `bc1`, `bc2`, `bc3` and `bc4`. The "translated" 16-bp cell barcodes in the formatted fastq file consists of four 4-bp barcodes, each with 96 unique sequences. Therefore, it is reasonable for us to assume the translation happens in a one-to-one manner, like shown below:

![](/images/2024-06-11/pipseeker_translation_scheme.png)

If we keep checking some reads, eventually we would figure out the dictionary for the translation. Let's check all reads, comparing the original ones to their formatted counterparts, in a more systematic way. To this end, we first do the following processing using the command lines[^1]:

```bash
join -1 1 -2 1 \
    <(zcat barcode/barcoded_fastqs/barcoded_*_R1.fastq.gz | paste - - - - | cut -f 1,2 | sort -k1,1) \
    <(zcat fastqs/RK20220802_FR_3_S3_L00*_R1_001.fastq.gz | paste - - - - | cut -f 1,2 | sort -k1,1) \
    > mapping.txt
```

If you are not familiar with this type of bash command, here are some simple explanations. First, the

```bash
join -1 1 -2 1 file1 file2 > mapping.txt
```

part says that use the first column of `file1` (`-1 1`) and the first column of `file2` (`-2 1`) as the join keys. Merge the line from the two files side by side if the join keys are matched. Redirect the joined output to new text file called `mapping.txt`. Then `file1` and `file2` is essentially `<(some commands)`, which means the `join` commands reads standard input, instead of actual files, from whatever comes out from the `some commands` in `<(some commands)`. Finally, the `paste - - - -` in the parentheses output four lines from the original output into one line. In this case, each line after the `paste` command would be one read. The `sort -k1,1` command basically sorts the read by read name (column 1), which is required by the `join` command.

Here is the top 10 lines from `mapping.txt`:

```bash
@VH00284:1:AAANWCCHV:1:1101:10013:11715 1:N:0:GGACTCCT+AGAGGATA AACAAGATCACCAGGCTATGTATTGCGT 1:N:0:GGACTCCT+AGAGGATA AAACTACAATGTTTCTCGAGGAAGAATCGAGCTCAAACATATGTATTGCGT
@VH00284:1:AAANWCCHV:1:1101:10013:12359 1:N:0:GGACTCCT+AGAGGATA ATCACCCCCAGTACAAGGTCATTTTGCA 1:N:0:GGACTCCT+AGAGGATA GTGAACTCATGCCAAATGAGATCAACTCGAGAGCATGCCGGTCATTTTGCA
@VH00284:1:AAANWCCHV:1:1101:10013:12548 1:N:0:GGACTCCT+AGAGGATA ACGTAAGAAGCTATCTGTTGATTCTCAA 1:N:0:GGACTCCT+AGAGGATA CCTATTTAATGTAGCGAGAGCTTGACTCGAGCAAGGTACGTTGATTCTCAA
@VH00284:1:AAANWCCHV:1:1101:10013:12965 1:N:0:GGACTCGT+AGAGGATA ACAGAGCGATTAACCGAGGCTGTATTCG 1:N:0:GGACTCGT+AGAGGATA CTGTTTCCATGGGTCTAGAGGAACAGTCGAGAGTAATGGAGGCTGTATTCG
@VH00284:1:AAANWCCHV:1:1101:10013:13154 1:N:0:GGACTCCT+AGAGGATA AAACAACAACAAACCTTAAGATAGGTAT 1:N:0:GGACTCCT+AGAGGATA CCTTTACAATGGGTTTCGAGTGGGTTTCGAGTCTAATTGTAAGATAGGTAT
@VH00284:1:AAANWCCHV:1:1101:10013:14555 1:N:0:GGACTCCT+AGAGGATA AATTAATCCACGCCTCAGCCCAGGTAGC 1:N:0:GGACTCCT+AGAGGATA CCACCTCTATGAAAGTGGAGACAAAGTCGAGACATGGACAGCCCAGGTAGC
@VH00284:1:AAANWCCHV:1:1101:10013:14858 1:N:0:GGACTCCT+AGAGGATA CCAGACAAAACGACCAGAATCAGCGCTG 1:N:0:GGACTCCT+AGAGGATA AATATGACATGAATAGCGAGGGTTTCTCGAGAATAAGGAGAATCAGCGCTG
@VH00284:1:AAANWCCHV:1:1101:10013:14934 1:N:0:GGACTCCT+AGAGGATA ATTACCTCCCTAAGCTCTACCGCTGCAT 1:N:0:GGACTCCT+AGAGGATA AAAGAGGCATGGCTCTTGAGATCTTCTCGAGGGTTAGGGCTACCGCTGCAT
@VH00284:1:AAANWCCHV:1:1101:10013:15312 1:N:0:CGACTCCT+AGAGGATA CCCCAGTGAATCCCCGACAGTTGGTGAG 1:N:0:CGACTCCT+AGAGGATA CTAACGCCATGTAACCCGAGTAGAACTCGAGCAAGGGTTACAGTTGGTGAG
@VH00284:1:AAANWCCHV:1:1101:10013:16070 1:N:0:GGACTCCT+AGAGGATA CAAGCAGGCAGGATTACAAGGTAGGTTG 1:N:0:GGACTCCT+AGAGGATA CCATCCACATGGCTAAGGAGGCACTATCGAGTTTGCCAGCAAGGTAGGTTG
```

As you can see, we paired the formatted reads with the original ones side-by-side. Now we could compare the formatted and original sequences within the pair. That is, the 3rd and the 5th columns (space-delimited). We could write a `python` script to do this. Note that everything is done under the `fluentbio` working directory:

```python
from collections import defaultdict

wl1 = [line.strip() for line in open('./fb_v3_bc1.tsv')]
wl2 = [line.strip() for line in open('./fb_v3_bc2.tsv')]
wl3 = [line.strip() for line in open('./fb_v3_bc3.tsv')]
wl4 = [line.strip() for line in open('./fb_v3_bc4.tsv')]

translate1 = {}
translate2 = {}
translate3 = {}
translate4 = {}

for w,t in zip([wl1, wl2, wl3, wl4], [translate1, translate2, translate3, translate4]):
    for i in w:
        t[i] = defaultdict(int)

with open('mapping.txt') as fh:
    for line in fh:
        _, _, bcout, _, bcin = line.strip().split(' ') # we only care about the 3rd and 5th columns
        key1 = bcin[:8]
        key2 = bcin[11:17]
        key3 = bcin[20:26]
        key4 = bcin[31:39]
        out1 = bcout[0:4]
        out2 = bcout[4:8]
        out3 = bcout[8:12]
        out4 = bcout[12:16]
        if key1 in wl1: # only record when key1 is in the white list of bc1
            translate1[key1][out1] += 1
        if key2 in wl2: # only record when key2 is in the white list of bc2
            translate2[key2][out2] += 1
        if key3 in wl3: # only record when key3 is in the white list of bc3
            translate3[key3][out3] += 1
        if key4 in wl4: # only record when key4 is in the white list of bc4
            translate4[key4][out4] += 1
```

The four dictionaries `translate{1..4}` contain the PIPseeker$^{\textmd{TM}}$. If the software is indeed using a one-to-one translation, we should expect in `translate{1..4}` that each original barcode in the whitelist should only have one entry or one dominating entry. It turned out that this is the case. Here is a screenshot of the first 10 entries in `translate1`:

![](/images/2024-06-11/v3_translate1.png)

I have now organised all four translations into `csv` files, and you can download them from the [scg_lib_structs](https://github.com/Teichlab/scg_lib_structs/tree/master/data/PIP-seq) GitHub repo.

## The V4 data

Since we know that the V4 chemistry uses the same whitelist as the V3 chemistry, there are just some variable bases at the beginning of Read 1 to desync the Illumina sequencing cycles[^2]. Therefore, it is reasonable for use to assume that the translation would remain the same. Is that the case? Let's find out.

First, we downloaded the [T20 V4 Mouse Brain Nuclei](https://www.fluentbio.com/t20-brain-nuclei/) data from **FluentBio**. We used the exact same procedures to get the `mapping.txt` file. Then a slightly different `python` script was needed to figure out the translation. Due to the variable bases at the beginning of Read 1, we used the built-in regular expression module `re`. The content of the script looks like this:

```python
import re
from collections import defaultdict

# compile the pattern with groups
pattern = re.compile(r'([ACGTN]{8})ATG([ACGTN]{6})GAG([ACGTN]{6})TCGAG([ACGTN]{8})')

wl1 = [line.strip() for line in open('../fb_v3_bc1.tsv')]
wl2 = [line.strip() for line in open('../fb_v3_bc2.tsv')]
wl3 = [line.strip() for line in open('../fb_v3_bc3.tsv')]
wl4 = [line.strip() for line in open('../fb_v3_bc4.tsv')]

translate1 = {}
translate2 = {}
translate3 = {}
translate4 = {}

for w,t in zip([wl1, wl2, wl3, wl4], [translate1, translate2, translate3, translate4]):
    for i in w:
        t[i] = defaultdict(int)

with open('mapping.txt') as fh:
    for line in fh:
        _, _, bcout, _, bcin = line.strip().split(' ')
        res = pattern.search(bcin)
        if not (res is None): # only look for matching records
            key1, key2, key3, key4 = res.groups()
            out1 = bcout[0:4]
            out2 = bcout[4:8]
            out3 = bcout[8:12]
            out4 = bcout[12:16]
            if key1 in wl1:
                translate1[key1][out1] += 1
            if key2 in wl2:
                translate2[key2][out2] += 1
            if key3 in wl3:
                translate3[key3][out3] += 1
            if key4 in wl4:
                translate4[key4][out4] += 1
```

Here is a screenshot of the first 10 entries in `translate1`:

![](/images/2024-06-11/v4_translate2.png)

As you can see that this data have some noise there. However, each entry in the original whitelist only have one dominating 4-mer after the translation. The top dominating one is orders of magnitude higher than the rest, so I guess it is safe to just use the 4-mer with the highest count.

Eventually, it turns out that the translation remains unchanged between the V3 and V4 chemistries.

[^1]: Not sure if this the most efficient way of achieving this purpose. If you have better ideas, please let me know.

[^2]: Check [the previous post](/posts/2023-07-02-pip-seq-v4-update/) where I talked about the V4 library structure.
---
title: "Guessing Library Structures (4) - PIP-seq V1prototype"
date: 2023-06-07T00:30:00+08:00
tags: ['single cell genomics', 'sequencing library', 'illumina sequencing', 'fluentbio', 'pipseq', 'seqspec']
draft: false
showTableOfContents: true
type: post
---

{{< note title="Portal" >}}
- [Guessing Library Structures (1) - Background](/posts/2023-05-28-guessing-library-structures-1/)
- [Guessing Library Structures (2) - PIPseq$^{\\textmd{TM}}$ V3](/posts/2023-06-03-guessing-library-structures-2/)
- [Guessing Library Structures (3) - PIP-seq V2](/posts/2023-06-04-guessing-library-structures-3/)
- [Guessing Library Structures (4) - PIP-seq V1prototype](/posts/2023-06-07-guessing-library-structures-4/)
{{< /note >}}

This is the fourth post, also the last, from a series where I use the recent [PIP-seq](https://www.nature.com/articles/s41587-023-01685-z) data as an example to demonstrate the importance to use a common standard like [*seqspec*](https://www.biorxiv.org/content/10.1101/2023.03.17.533215v1) to accompany sequencing reads submissions into public repositories.

## The V1? Chemistry

The mysterious samples come from the experiments of looking at the single-cell transcriptional responses of two cancer cell lines (H1975 and PC9) to gefitinibis. I name it **"PIP-seq V1prototype"**, which by no means is accurate. Anyway, they are:

```bash
GSM6106784 Fresh 2000 cell H1975 with drug treatment
GSM6106785 Fresh 2000 cell H1975 with control DMSO treatment
GSM6106786 Fresh 2000 cell PC9 with drug treatment
GSM6106787 Fresh 2000 cell PC9 with control DMSO treatment
GSM6106788 Fresh 1000 cell PC9:H1975 9:1 with drug treatment
```

Here, we are just using `SRR9086119` as an example. We started with downloading the FASTQ files, and have a look at the content inside:

```bash
$ fastq-dump --split-3 -A SRR19086119

$ zcat SRR19086119_1.fastq.gz | head -12
@SRR19086119.1 1 length=35
NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
+SRR19086119.1 1 length=35
###################################
@SRR19086119.2 2 length=51
TGAATNCGACGGAGTGATTGCTTGTGACGCCTTACGTGTTTCCTCCATTTT
+SRR19086119.2 2 length=51
AAAAA#EAAEEAEEAA<AEEEEAEEEEEEEEE6EEEEEE/AEEEEEEEEEE
@SRR19086119.3 3 length=51
GATAANCATCGAGTGATTGCTTGTGACGCCTTGAGTTCACTCCCGGTTTTT
+SRR19086119.3 3 length=51
A/AAA#EEEEE/EEEEEEEEAEEAAEEEEEEEEEEEEAEAEEEEE<EEEEE
```

It seems the reads are variable in length, and it seems 51 bp is the longest. I'm not entirely sure what is going on here, since I do not have much experience with GEO submission. Let's focus on the reads whose lengths are 51 bp which are more likely to be the untrimmed good reads. We can do this by:

```bash
zcat SRR19086119_1.fastq.gz | \
    grep 'length=51' -A 1 | \
    sed '/^--$/d' | \
    gzip > SRR19086119_1_51bp.fastq.gz
```

Now we could start FastQC with the new FASTQ:

![](/images/2023-06-07/pip-seq_srr19086119.png)

Now we do not immediately see constant regions like linkers or splints. According to the experience described in the previous posts, we think there should be some linkers or splints in the reads. It is likely that there are some variable bases in the reads to increase base complexities in each sequencing cycle. Whenever this happens, we could print 40 sequences (only showing 10 below), which is the number of sequences that fit well with the height of my screen:

```bash
$ zcat SRR19086119_1_51bp.fastq.gz | sed -n '2~4 p' | head -40
TGAATNCGACGGAGTGATTGCTTGTGACGCCTTACGTGTTTCCTCCATTTT
GATAANCATCGAGTGATTGCTTGTGACGCCTTGAGTTCACTCCCGGTTTTT
TGAGGNATAGAGAGTGATTGCTTGTGACGCCTTGCCTCTTTTCGCCGTTTT
GAAAGNTTGTGAGTGATTGCTTGTGACGCCTTGATGTATTAGGGCTTTTTT
GATGTNCCAGGAGTGATTGCTTGTGACGCCTTAATGTTTGACGTATTTTTT
AATGTNTGGAGTGATTGCTTGTGACGCCTTCTAGTAACGTTTGATTTTTTT
ACCCANGAGAGTGATTGCTTGTGACGCCTTGTTACCGCATGCTCTTTTTTT
TGAAGNGTAGGGAGTGATTGCTTGTGACGCCTTCTATAGAGTGTGCATTTT
GACCANATTAGAGTGATTGCTTGTGACGCCTTAGCTACGGATGAGTTTTTT
GCTCTNGTGAGTGATTGCTTGTGACGCCTTAGGAAATCAGGCCATTTTTTT
```

Then, we could copy and paste those sequences into [**VS Code**](https://code.visualstudio.com), and start select some strings with our cursor around the middle of the reads. When you start selection, VS Code automatically highlights all occurrences of your selection in the current file, like shown below (only 10 sequences are shown):

![](/images/2023-06-07/vscode_selection.png)

By playing around with this and using the multi-line editing function, also bear in mind that the variable regions flanking the constant linker or splint contain cell barcode and UMI information, we can separate the reads like this:

```bash
TGA ATNCGACG GAGTGATTGCTTGTGACGCCTT ACGTGTTTCCTCCA TTTT
 GA TAANCATC GAGTGATTGCTTGTGACGCCTT GAGTTCACTCCCGG TTTTT
TGA GGNATAGA GAGTGATTGCTTGTGACGCCTT GCCTCTTTTCGCCG TTTT
 GA AAGNTTGT GAGTGATTGCTTGTGACGCCTT GATGTATTAGGGCT TTTTT
 GA TGTNCCAG GAGTGATTGCTTGTGACGCCTT AATGTTTGACGTAT TTTTT
    AATGTNTG GAGTGATTGCTTGTGACGCCTT CTAGTAACGTTTGA TTTTTTT
    ACCCANGA GAGTGATTGCTTGTGACGCCTT GTTACCGCATGCTC TTTTTTT
TGA AGNGTAGG GAGTGATTGCTTGTGACGCCTT CTATAGAGTGTGCA TTTT
 GA CCANATTA GAGTGATTGCTTGTGACGCCTT AGCTACGGATGAGT TTTTT
    GCTCTNGT GAGTGATTGCTTGTGACGCCTT AGGAAATCAGGCCA TTTTTTT
    GCTTTNGC GAGTGATTGCTTGTGACGCCTT GGAAACAGCCATTG TTTTTTT
    GCAGTNGA GAGTGATTGCTTGTGACGCCTT GGGCCAATGAAGTG TTTTTTT
 GA AAGNTTGT GAGTGATTGCTTGTGACACCTT CGAACGTATCGTTC TTTTT
  A AGCTNCGG GAGTGATTGCTTGTGACGCCTT AAACAGGGTCATCC TTTTTT
    TCCTTNTT GAGTGATTGCTTGTGACGCCTT ACCACGCTAAAAAA TTTTTTT
    AACTTNGC GAGTGATTGCTTGTGACGCCTT AAGTTTCGAAGTTT TTTTTTT
    GAGTCNAG GAGTGATTGCTTGTGACGCCGT ACTCACCGGGAGCG TTTTGTT
    TCCAGNGA GAGTGATTGCTTGTGACGCCTT GGCCTAAGTGCAAT TTTTTTT
    GGGTTNGT GAGTGATTGCTTGTGACGCCTT AACCCTTGGCTACT TTTTTTT
 GA GACNATGG GAGTGATTGCTTGTGACGCCTT CGTACCTAACCATT TTTTT
TGA GANGCACT GAGTGATTGCTTGTGACGCCTT TCGGTTTACCCAAT TTTT
 GA GGCNTTAG GAGTGATTGCTTGTGACGCCTT AGGTTGGTAACATC TTTTT
 GA TCGNTACG GAGTGATTGCTTGTGACGCCTT GTTCCAGACTTACT TTTTT
 GA GAGNCCAT GAGTGATTGCTTGTGACGCCTT AGCTCCGTACCGTG TTTTT
TGA ATNACCGA GAGTGATTGCTTGTGACGCCTT ACCAAGATGTGAAG TTTT
  A TCACNTTT GAGTGATTGCTTGTGACGCCTT ACCTTCTTGGAATC TTTTTT
 GA CTTNTTCG GAGTGATTGCTTGTGACGCCTT AGCAGAACACCTTA TTTTT
 GA CTCNTGAC GAGTGATTGCTTGTAACGCCTT GCCACATCAGCGAC TTTTT
    TACCGNCA GAGTGATTGCTTGTGACGCCTT ATGGAAATCTGAAC TTTTTTT
 GA GGCGTTAG GAGTGATTGCTTGTGACGCCTT AGGCAACGCTTTCA TTTTT
    TGCAAGGG GAGTGATTGCTTGTGACGCCTT GCCACATCAGTGCC TTTTTTT
  A TAGTCGCA GAGTGATTGCTTGTGACGCCTT ATGATCTATTTGCG TTTTTT
 GA CATTTGTT GAGTGATTGCTTGTGACGCCTT ATGGAGCTGCTGCA TTTTT
TGA ACATCTAT GAGTGATTGCTTGTGACGCCTT ACAGCGGAAAGTCG TTTT
 GA TTGATCTA GAGTGATTGCTTGTGACGCCTT ACCCATATTGCCAT TTTTT
  A ATGGATTA GAGTGATTGCTTGTGACGCCTT TTGCACGCTTGGTA TTTTTT
  A GCACCTCT GAGTGATTGCTTGTGACGCCTT TGATGCCAAGATTC TCATTT
    AATGTTTG GAGTGATTGCTTGTGACGCCTT CTAGTACCGTTTGA TTTTTTT
TGA CAACAAAT GAGTGATTGCTTGTGACGCCTT GAATACTTACCCTA TTTT
 GA TGTAGTTT GAGTGATTGCTTGTGACGCCTT AATTGCGATTATTA TTTTT
```
Now it seems the sequence of the constant region is `GAGTGATTGCTTGTGACGCCTT`, which is basically the so called **W1** linker sequence in [inDrop](https://teichlab.github.io/scg_lib_structs/methods_html/inDrop.html). Before W1, there are 8 - 11 bp sequence. We also noticed that when the leading sequence is 9 bp in length, it always started with `A`; if the leading sequence is 10 bp in length, it always started with `GA`; if the leading sequence is 11 bp in length, it always started with `TGA`. Therefore, the variable bases are `None/A/GA/TGA`. The 8 bp after the variable bases should be barcodes. The 14 bp after W1 should be barcodes and UMI, just like **inDrop**.

### The Percentage of Reads With The Correct Pattern

Now let's confirm our guesses. First, let's check the percentage of reads containing the patterns:

```bash
zcat SRR19086119_1_51bp.fastq.gz | \
    grep -P '[ACGTN]{8,11}GAGTGATTGCTTGTGACGCCTT[ACGTN]{14}T*' | wc -l
```

The output is 16,629,904. The FASTQ file contains 18,371,831 reads. Therefore, the percentage of reads with the correct pattern is $16629904/18371831 = 90.52\\%$. This is very promising. Note `grep` only returns perfect matches. Therefore if we allow some mismatches, the percentage would be even higher.

### The Beginning Variable Bases

Now let's see if the variable bases at the beginning of the reads are indeed `None/A/GA/TGA`:

```bash
zcat SRR19086119_1_51bp.fastq.gz | \
    grep -P '[ACGTN]{8,11}GAGTGATTGCTTGTGACGCCTT[ACGTN]{14}T*' | \
    awk -F 'GAGTGATTGCTTGTGACGCCTT' '{print $1}' | \
    rev | cut -c 9- | rev | \
    sort | uniq -c | \
    sort -b -k1,1nr
```

The results (only showing top 10 lines) are:

```bash
4367189 GA
4158045
3971934 A
3928682 TGA
  62870 G
  30395 TG
  13505 TA
   8686 AA
   7645 T
   6857 AGA
```

As you can see, the most frequent bases are `GA`, `None`, `A` and `TGA`. They are orders of magnitude more than all of the rest.

### The 8-bp Barcode Before W1

Now let's see if the 8-bp sequences before W1 might be barcodes. If they are, there should be a fixed number (like 48, 96, 384 *etc*.) of 8-mers. We use similar commands as described before:

```bash
zcat SRR19086119_1_51bp.fastq.gz | \
    grep -P '[ACGTN]{8,11}GAGTGATTGCTTGTGACGCCTT[ACGTN]{14}T*' | \
    awk -F 'GAGTGATTGCTTGTGACGCCTT' '{print $1}' | \
    rev | cut -c 1-8 | rev | \
    sort | uniq -c | \
    sort -b -k1,1nr |
    less -N
```

The above commands display the counts of all possible 8-mers in the 8 bp immediately before the W1 sequence, sorted in descending order. If we check the output, the occurrences of the top 383 8-mers are comparable, but there is a sudden drop from 383rd to 384th 8-mer. The difference is quite sharp. Below are lines 381 - 385:

{{< highlight bash "linenos=table,hl_lines=3-4,linenostart=381" >}}
17125 CTTACTCC
15511 GGTCTGAC
15179 GTCAATAC
 2574 GAAAGCCC
 1301 AATGCCCC
{{< / highlight >}}

It seems 383 barcodes are used here for the first barcodes.

### The Barcode And UMI After W1

Okay, so this is where I get stuck. It seems I could not find a fixed set of barcodes within the 14 bp after W1. I tried to look at different positions without success. Since this is similar to **inDrop**, I assume the 14 bp after W1 contains 8 bp barcodes + 6 bp UMI. However, the 8-bp barcodes here do not have a fixed set of sequences. Therefore, I could not get a whitelist.

The proposed library structure is [here](https://teichlab.github.io/scg_lib_structs/methods_html/PIP-seq_v1p.html), but this is not really correct. I will update once we get more information in the future.

## Conclusion

It really took a long time to guess and figure out the library structure of a new method. Very often, just reading the method section is not sufficient. We have to do some digging with the data. As you can see, even after messing around with the FASTQ files for quite a long time, there is still something we have not really figured out. In the future, the whole painful procedures can be completely avoided by providing a [*seqspec*](https://www.biorxiv.org/content/10.1101/2023.03.17.533215v1) together with the sequencing reads.
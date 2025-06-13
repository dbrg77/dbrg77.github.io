---
title: "Getting Index Reads From SRA"
date: 2022-07-26T10:07:27+08:00
tags: ['bioinformatics', 'fastq', 'illumina-sequencing', 'sratools', 'sequencing library', 'single cell genomics']
type: post
---

A few month ago, I ranted about the [__FastQ files submission__](/posts/2022-02-27-about-fastq-files-submission/) for data produced by complex genomic protocols. The main reason of the rant is that we tried to get some single cell data from [__SRA__](https://www.ncbi.nlm.nih.gov/sra), and we really needed the cell barcodes for the data preprocessing and analysis. However, the cell barcodes are stored in the indexing reads where it _seems_ impossible to get from the `.sra` file. Recently, I finally found a way of getting those information ... well ... at least for some studies. Here, I'm just documenting what happened.

First, the raw sequencing data is usually stored in the `sra` format, and you normally need the `fastq` files to start any analysis. To this end, you need the `fastq-dump` or `fasterq-dump` utilities from the [__sratools__](https://github.com/ncbi/sra-tools), which is not really well documented and contains many useful tricks that are obscure to non-routine users. For many studies, the sequencing is done using a pair-end mode, and you want to get both reads (`R1` and `R2`) in the fastq format. Let's start with using `fasterq-dump` as it is a more recent and, as the name suggested, a faster version of the previously used `fastq-dump` program. If you look at the help message of `fasterq-dump` you see two options for the purpose of getting paired fastq files:

```
-S|--split-files      write reads into different files
-3|--split-3          writes single reads into special file
```

This is not very helpful. Luckily, I have used the old `fastq-dump` in the old days and seem to remember it has a better description. This is the help message from `fastq-dump`:

```
--split-files           Write reads into separate files. Read
                          number will be suffixed to the file
                          name. NOTE! The `--split-3` option is
                          recommended. In cases where not all
                          spots have the same number of reads,
                          this option will produce files that WILL
                          CAUSE ERRORS in most programs which
                          process split pair fastq files.
--split-3               3-way splitting for mate-pairs. For each
                          spot, if there are two biological reads
                          satisfying filter conditions, the first
                          is placed in the `*_1.fastq` file, and
                          the second is placed in the `*_2.fastq`
                          file. If there is only one biological
                          read satisfying the filter conditions,
                          it is placed in the `*.fastq` file.All
                          other reads in the spot are ignored.
```

Okay, now it seems they recommend the `--split-3` option, so I guess I will use `fasterq-dump` with the `--split-3` flag, which is what I do on a regular basis due to the recommendation in the help message. However, according to the help message, only `*_1.fastq` and `*_2.fastq` files will be kept, and all other reads are ignored. In this case, "other reads" means "technical reads" which actually contain the index reads (cell barcodes) we want. Luckily, `fasterq-dump` has a `--include-technical` option. Thank Jon Trow from NLM Support helpdesk for letting me know this option. This should be good for us. Now let's do it using `SRR9672090` from [__SNARE-seq__](https://www.nature.com/articles/s41587-019-0290-0) as an example, which is the ATAC modality from the mouse cortex experiments:

```
$ fasterq-dump --split-3 --include-technical SRR9672090
spots read      : 124,642,473
reads read      : 373,927,419
reads written   : 249,284,946
technical reads : 124,642,473
```

However, only two files, `SRR9672090_1.fastq` and `SRR9672090_2.fastq`, were generated, each with 124,642,473 reads. I don't know where the index reads (technical reads) are. Maybe I should try the not-recommended `--split-files` option:

```
$ fasterq-dump --split-files --include-technical SRR9672090
spots read      : 124,642,473
reads read      : 373,927,419
reads written   : 373,927,419
```

This time, three files `SRR9672090_1.fastq`, `SRR9672090_2.fastq` and `SRR9672090_3.fastq`, were generated, each with 124,642,473 reads. Great! This is what I want. The `SRR9672090_1.fastq` and `SRR9672090_2.fastq` are the actual ATAC fragment reads, and the `SRR9672090_3.fastq` contains the cell barcodes.

For the SNARE-seq data, now I actually learnt from Tim Stuart's [__blog post__](https://timoast.github.io/blog/sra-aws/) and [__github repository__](https://github.com/timoast/SNARE-seq) that you can directly download the user submitted fastq files. However, I cannot download via the URLs (due to geographical resitrctions ???).

Anyway, now let's try `SRR1947692`, which is the HEK293T and GM12878 mixture experiment from [__sci-ATAC-seq__](https://www.science.org/doi/10.1126/science.aab1601):

```
$ fasterq-dump --split-files --include-technical SRR1947692
spots read      : 20,027,825
reads read      : 40,055,650
reads written   : 40,055,650
```

Sadly, only two files `SRR1947692_1.fastq` and `SRR1947692_2.fastq` were generated, each with 20,027,825 reads. This is kind of expected, because in the early days of single cell genomics, it was a common pratice to put cell barcodes into the fastq heeader, and not to inlcude them as separate fastq files. Now let's just look at the fastq header:

```bash
$ awk 'NR%4==1' SRR1947692_1.fastq | head
@SRR1947692.1 SHEN-MISEQ02:1:1101:15311:1341 length=51
@SRR1947692.2 SHEN-MISEQ02:1:1101:16462:1350 length=51
@SRR1947692.3 SHEN-MISEQ02:1:1101:16204:1351 length=51
@SRR1947692.4 SHEN-MISEQ02:1:1101:16257:1355 length=51
@SRR1947692.5 SHEN-MISEQ02:1:1101:15016:1376 length=51
```

Oh well ... not so useful, right? Then, I saw this documentation from the [__MAESTRO website__](https://baigal628.github.io/MAESTRO_documentation/sci_atac.html), where they used the following command to get the fastq files with proper headers:

```bash
fastq-dump --split-files --origfmt --defline-seq '@rd.$si:$sg:$sn' SRR1947692
```

It seems you could preserve the fastq header information in its originally submitted format with the `--origfmt` option, and use `--defline-seq` to specify the exact format. The `$sg` is the `spot-group`, which often contains the index information. Then, I think I can just use `fasterq-dump` for faster processing. Since I knew that there are no technical reads in this case, so I did not include that flag. However, I got an error message:

```bash
$ fasterq-dump --split-files --origfmt --defline-seq '@rd.$si:$sg:$sn' SRR1947692
unrecognized option: '--origfmt'
```

It turns out that the option `--origfmt` [__is not supported anymore__](https://github.com/ncbi/sra-tools/issues/233) in `fasterq-dump`. Let's remove it to see what happens:

```bash
$ fasterq-dump --split-files --defline-seq '@rd.$si:$sg:$sn' SRR1947692
unrecognized option: '--defline-seq'
```

That was just brilliant ... By invoking the `fasterq-dump` help message, we know that we should use `--seq-defline` instead:

```bash
$ fasterq-dump --split-files --seq-defline '@rd.$si:$sg:$sn' SRR1947692
spots read      : 20,027,825
reads read      : 40,055,650
reads written   : 40,055,650

$ awk 'NR%4==1' SRR1947692_1.fastq | head -5
@rd.1::SHEN-MISEQ02:1:1101:15311:1341
@rd.2::SHEN-MISEQ02:1:1101:16462:1350
@rd.3::SHEN-MISEQ02:1:1101:16204:1351
@rd.4::SHEN-MISEQ02:1:1101:16257:1355
@rd.5::SHEN-MISEQ02:1:1101:15016:1376
```

Oh, it seems without the `--origfmt`, `$sg` will not be written in the output (Note there is no content between the first and second colons in the header). I should have just used the exact the same command in the MAESTRO website:

```bash
$ fastq-dump --split-files --origfmt --defline-seq '@rd.$si:$sg:$sn' SRR1947692
Read 20027825 spots for SRR1947692
Written 20027825 spots for SRR1947692

$ awk 'NR%4==1' SRR1947692_1.fastq | head -5
@rd.1:TCTCCCGCCGAGGCTGACTGCATAAGGCGAAT:SHEN-MISEQ02:1:1101:15311:1341
@rd.2:CGCGTTCCAGGCCGTACCTGCATAAGGCGACG:SHEN-MISEQ02:1:1101:16462:1350
@rd.3:GAGATTCCCGTACTAGAAGGAGTATATAGCCT:SHEN-MISEQ02:1:1101:16204:1351
@rd.4:GAGATTCCCGTACTAGGTAAGGAGATAGAGGC:SHEN-MISEQ02:1:1101:16257:1355
@rd.5:ATTACTCGAAGAGGCACGAGTAGACAGGACGT:SHEN-MISEQ02:1:1101:15016:1376
```

Ha, I'm so happy ... You see the 32-bp DNA sequences between the first and the second colons in the fastq header? Those are the cell barcodes, each of which is a combination of `Tn5-T5` (8 bp), `Tn5-T7` (8 bp), `Nextera i5` (8 bp) and `Nextera i7` (8 bp). Check [__this page__](https://teichlab.github.io/scg_lib_structs/methods_html/sci-ATAC-seq.html) for more information.

Finally, for the data that DO have technical reads available like SNARE-seq, what if you want both the technical reads and the `$sg` information? Well ... I'm afraid that you cannot use `fasterq-dump`. You have to do this in a slower way using `fastq-dump`. The following command will give you three fastq files, with the sample index in the fastq header.

```bash
$ fastq-dump --split-files --origfmt --defline-seq '@rd.$si:$sg:$sn' SRR9672090
Read 124642473 spots for SRR9672090
Written 124642473 spots for SRR9672090

$ awk 'NR%4==1' SRR9672090_3.fastq | head -5
@rd.1:TATCCTCT:D00611:697:CD0V6ANXX:5:2301:1176:2478
@rd.2:TATCCTCT:D00611:697:CD0V6ANXX:5:2301:1480:2408
@rd.3:TATCCTAT:D00611:697:CD0V6ANXX:5:2301:1361:2447
@rd.4:TATCCTCT:D00611:697:CD0V6ANXX:5:2301:1384:2495
@rd.5:TATCCTCT:D00611:697:CD0V6ANXX:5:2301:1335:2498
```

In this case, you don't need to specify `--include-technical` because it will not work with `fastq-dump`. The `--split-files` flag will dump technical reads if there is any. Anyway, I hope this is useful to people who do not use sratools very often, like me.
---
title: "About FastQ Files Submission"
date: 2022-02-27T10:07:27+08:00
tags: ['bioinformatics', 'fastq', 'illumina-sequencing', 'sequencing library', 'single cell genomics']
type: post
---

This post collects my thoughts about `fastq` files accumlated over the years, specifically focusing on what should be included during a submission. It is not about the format of fastq files. If you are not sure about the format, you can check the [Wikipedia page](https://en.wikipedia.org/wiki/FASTQ_format) and this excellent paper by __[Cock _et al._](https://doi.org/10.1093/nar/gkp1137)__ that contains very deteiled information and the history about the fastq file.

Let's first look at what you will get from a typical Illumina sequencing experiment. The library fragments you get is like this:

![](/images/2022-02-27/lib_struct.pdf)

The middle dark thin lines are the DNA you want to sequence. The coloured boxes represent known sequences, often called adaptors, designed by Illumina. __[Click here](https://teichlab.github.io/scg_lib_structs/methods_html/Illumina.html)__ to look at the actual sequences if you are new to Illumina sequencing. The indices (`i5` and `i7`) shown above are short unique DNA sequences, often called barcodes, that will discrimate different samples. Those allow mixing and sequencing libraries from different samples at the same time. Now let's start the sequencing with a typical dual index pair end mode:

__Step 1__ is to denature the DNA, use the bottom strand as the template and add the `Read 1 primer` to start sequencing your DNA in the middle. This step yields a file called `Read1.fastq.gz` (note: striclty speaking, this is not the case, see the later section for the explanation):

![](/images/2022-02-27/read1.pdf)

__Step 2__ is to use the bottom strand as the template again, add the `Index 1 primer` that is reverse complementary to the `Read 2 primer` and sequence the `i7` index. This step yields a file called `I1.fastq.gz`:

![](/images/2022-02-27/index1.pdf)

__Step 3__ is to perform the cluster regeneration, use the top strand as the template, add the `Index 2 primer` that is reverse complementary to the `Read 1 primer` and sequence the `i5` index. This step yields a file called `I2.fastq.gz`:

![](/images/2022-02-27/index2.pdf)

__Step 4__ is to use the top strand as the template again and add the `Read 2 primer` to sequence the other end of your DNA. This step yields a file called `Read2.fastq.gz`.

![](/images/2022-02-27/read2.pdf)

The real situation is bit more complicated. You often only have one index (`i7`), so __Step 3__ is omitted. Sometimes, you only do single end sequencing, so __Step 4__ is omitted. In addition, the bases `A, C, G, T` are fluorescent signals. Those signals (images) are converted into DNA bases by a process called `base calling`, which is automatically done on the Illumina machine. The output you get directly from the sequencing machine is the result of the `base calling` step. The files are in the `BCL` (Binary Base Calling) format. To obtain the fastq files, you need to run the [bcl2fatq](https://support.illumina.com/sequencing/sequencing_software/bcl2fastq-conversion-software.html) program provided by Illumina to covnert the `bcl` file to the fastq format.

If you haven't done that before, your sequencing core facility is probably doing those steps for you. They probably also seperate different samples for you based on the index you provide, a process called `demultiplexing`. Eventually, you get `Read1.fastq.gz` and `Read2.fastq.gz` per sample. Those are the actual DNA sequences you care about, and those files contain the useful information about your experiments. If you are running `bcl2fastq` by yourself, you may only get two, not four, fastq files per sample that are `Read1.fastq.gz` and `Read2.fastq.gz`. This is due to the default behaviour of the `bcl2fastq` program. To get all four fastq files, add the `--create-fastq-for-index-reads` flag when running `bcl2fastq`.

In the past, `Read1.fastq.gz` and `Read2.fastq.gz` were all we need, because all the actual information of the experiments are there. The index files (`I1.fastq.gz` and `I2.fastq.gz`) are not really useful once the demultiplexing step is done. That's why people just submit `Read1.fastq.gz` and `Read2.fastq.gz` to a public repostory, such as [ArrayExpress](https://www.ebi.ac.uk/arrayexpress) and [GEO](https://www.ncbi.nlm.nih.gov/geo/), when they publish their papers. However, with the ever-growing complexity of current genomic methods, especially those in the field of single cell genomics, having just two fastq files is not enough.

In single cell genomics, the words `sample`, `index` and `barcode` are quite vague. Therefore, we now talk about the `cell barcode`, a short DNA sequence that is unique to a cell. All reads that are associated with a particular cell barcode are from the same cell. In any single cell methods, the `UMI`, if any, are usually in either `Read1.fastq.gz` or `Read2.fastq.gz`. However, the `cell barcode` can be anywhere in `Read1.fastq.gz`, `Read2.fastq.gz`, `I1.fastq.gz` and `I2.fastq.gz`. Sometime, a cell barcode is defined by the combination of sequences from those fastq files, this is especially true for all the "split-pool" based methods. To deal with this, I often put `N` in the `SampleSheet.csv`, a file that is used during the `bcl2fastq` step, like this:

```
# this is an example from NextSeq
Sample_ID,Sample_Name,Sample_Plate,Sample_Well,Index_Plate,Index_Plate_Well,I7_Index_ID,index,I5_Index_ID,index2,Sample_Project,Description
foo,,,,,,N001,NNNNNNNN,S501,NNNNNNNN,,
```

If you do that, `bcl2fastq` will look for literal `N` in your index due to a known bug of the program, and anything matching `NNNNNNNN+NNNNNNNN` will be written into the sample `foo`. Hopefully, you will have very few or no reads in `foo`. All the useful reads will be written into the `Undetermined` sample. You will get the following four files per run:


```
Undetermined_R1.fastq.gz
Undetermined_R2.fastq.gz
Undetermined_I1.fastq.gz
Undetermined_I2.fastq.gz
```

All the information is there, and you can do whatever you need from this point. For me, I like the flexible settings of [STARsolo](https://github.com/alexdobin/STAR/blob/master/docs/STARsolo.md), [kallisto|bustools](https://www.kallistobus.tools) and [zUMIs](https://github.com/sdparekh/zUMIs) _etc._ which allow preprocessing any methods in theory. For example, I don't really need to specify if the experiment protocol is using `10xv123`, `Drop-seq` or `CEL-seq` _etc._ I can use the combination of `soloCBstart`, `soloCBlen`, `soloUMIstart` and `soloUMIlen` in STARsolo to specify the locations of CB and UMI, even for future new methods. If you can't figure out the CB and UMI locations in a method, especially in a complicated single cell method, then can I shameless advertise my [scg_lib_structs](https://github.com/Teichlab/scg_lib_structs) repository to you 😉? Okay, I guess my real point is we need to submit all four fastq files to public repositories.
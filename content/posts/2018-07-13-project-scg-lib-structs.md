---
title: Project scg_lib_structs
date: 2018-07-13T00:00:00+08:00
tags: ['single cell genomics', 'sequencing library', 'illumina sequencing']
type: post
---

Being able to perform genomic studies on the material obtained from single cells is just awesome. In 2009, [__Tang__ ___et al___.](https://www.nature.com/articles/nmeth.1315) presented the first study of mRNA-seq from single cells (scRNA-seq). Two years later, [__Navin__ ___et al___.](https://www.nature.com/articles/nature09807) successfully sequenced the genome of single cells. Ever since then, the field of single cell genomics is rapidly evolving and has proved to be powerful in many aspects of biology.

Many methods (mainly for scRNA-seq) are invented along the way, which is a good thing. However, I'm a bit bombarded and lost in those methods. Many of them are full of technical jargons and it is not straightforward to figure out how they actually work and what the differences are among those methods. To help understand how a method works, people tend to present some schematic views of how the sequencing library is generated using their method. For example, this is a very nice summary of 8 different scRNA-seq methods taken from a recent review by [__Ziegenhain__ ___et al___.](https://academic.oup.com/bfg/advance-article/doi/10.1093/bfgp/ely009/4951519):

![](/images/000/schematic_view_of_libraries.png)

It is very clear and very helpful, but my main problem with this kind of schematic view is that it often omits many important details and does not help with troubleshooting. From my own experience, when I try a new scRNA-seq method, most likely I will fail in the first few attempts. After getting the sequencing data, I often use `grep` to have a look at the sequences from the fastq files, and `grep` is pretty good for troubleshooting genomic experiments. However, I still have a few things that I need to figure out:

- What are the adapter sequences to `grep`? i.e. the grey bar in the figure above.
- Since DNA is double stranded, should I `grep` the adapter sequences as they are, or should I do the reverse complement?
- Since paired end sequencing is often used, should I `grep` in Read 1 or Read 2?
- Reads often contain partial adapters, should I `grep` 5'- or 3'-end of the adapters or the reads?

The answers to those questions are not obvious by just looking at the schematic view of the library generation. What really helps is to draw from scratch the actual DNA sequences from the fist step (often reverse transcription) to the final library construction step (often PCR amplification). Supplementary Figure S1 from the [__SCRB-seq__](https://www.biorxiv.org/content/early/2014/03/05/003236) paper and the [__Technical Note from 10x Genomics__](https://support.10xgenomics.com/single-cell-gene-expression/library-prep/doc/technical-note-assay-scheme-and-configuration-of-chromium-single-cell-3-v2-libraries) are very good examples.

Therefore, to help myself understand all of the methods and future troubleshooting, I start the project `scg_lib_structs` (that stands for "Single Cell Genomics Library Structures"), an on-paper library preparation done by myself. Today, I have finally finished it (well ... for now ... will continue once new methods are invented), and it mostly contains scRNA-seq methods. My main aim is to create something simple and straightforward to see. To that end, I need the following requirements:

- Coloured text support for easy reading
- Flexible and custom spacing for sequence alignment
- Easily accessible

After messing around a bit, it seems the `<pre></pre>` tag from HTML fitted those requirements. Therefore, I decided to use HTML to write a webpage for each method with a plain text editor. It is kind of stupid, since I have to manually align the sequence, which involves heavy pressing of the <kbd>Space</kbd> and <kbd>⌫</kbd> buttons (you will see what I mean if you open the HTMLs with a text editor), but I don't know any other way of doing this.

There are a lot of repetitions during generating those pages, because the basic chemistry among them are not that different. We recently wrote a review which summarised the main differences of different scRNA-seq methods. You can check all the text boxes and Figure 2 from [__our review__](https://www.annualreviews.org/doi/10.1146/annurev-biodatasci-080917-013452). Not all methods were included in the review, since some are not published when we started to write the review. The `scg_lib_structs` project page has a more updated version of the comparison.

All files from this project is here: [__https://github.com/Teichlab/scg_lib_structs__](https://github.com/Teichlab/scg_lib_structs) 

GitHub Pages here: [__https://teichlab.github.io/scg_lib_structs/__](https://teichlab.github.io/scg_lib_structs/)

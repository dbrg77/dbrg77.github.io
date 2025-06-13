---
title: "Guessing Library Structures (1) - Background"
date: 2023-05-28T14:46:00+08:00
tags: ['single cell genomics', 'sequencing library', 'illumina sequencing', 'fluentbio', 'pipseq', 'seqspec']
draft: false
showTableOfContents: false
type: post
---

{{< note title="Portal" >}}
- [Guessing Library Structures (1) - Background](/posts/2023-05-28-guessing-library-structures-1/)
- [Guessing Library Structures (2) - PIPseq$^{\\textmd{TM}}$ V3](/posts/2023-06-03-guessing-library-structures-2/)
- [Guessing Library Structures (3) - PIP-seq V2](/posts/2023-06-04-guessing-library-structures-3/)
- [Guessing Library Structures (4) - PIP-seq V1prototype](/posts/2023-06-07-guessing-library-structures-4/)
{{< /note >}}

This is the first post from a series where I use the recent [PIP-seq](https://www.nature.com/articles/s41587-023-01685-z) data as an example to demonstrate the importance to use a common standard like [*seqspec*](https://www.biorxiv.org/content/10.1101/2023.03.17.533215v1) to accompany sequencing reads submissions into public repositories.

## What's The Problem?

There are many interesting questions we can ask, but to what extent can we investigate and do research on them? This is heavily dependent on the methods we have. We believe methods matter!

Very often, we have quite a few different methods to achieve the same goal (almost). This is especially true in the field of genomics. Just think about how many methods (or variations) there are for doing ChIP-seq, for performing peak calling, for doing RNA-seq and for performing differential expression analysis *etc*. In order to choose the methods that fit our purposes, we need to compare them for a specific application. For example, in single-cell RNA-seq (scRNA-seq), one often asks: for the same amount of money,

- what methods detect more genes?
- what methods produce more **usable reads**?

Whenever I talk about those method comparisons, I often get a certain type of responses such as **"method comparisons are boring"** or **"those are not biology"** *etc*. I used to get upset about this, but now I have heard enough of those and can comfortably ignore those comments. Anyway ... To answer those questions in a fair way, we need a uniform preprocessing pipeline for the methods we want to compare, which is **NOT** trivial at all.

We have seen so many tutorials on the downstream analysis of scRNA-seq data, such as dimensionality reduction, integration, batch correction, clustering *etc*. All of those tutorials start with a **cell-by-gene count matrix**. However, very few talk about how to obtain the count matrix from raw reads, that is, the FASTQ files. One reason is that many people use the kits from **10x Genomics**, and the count matrix can be easily obtained by [Cell Ranger](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/what-is-cell-ranger). This is fine, and we also use it. However, it is a proprietary software ([some versions](https://github.com/10XGenomics/cellranger/tags) are open sourced) that takes a lot of resource and time to run and is almost impossible to customise. Many tutorials assume you have the output from Cell Ranger or Cell Ranger-like output. I hope people do realise that there are [way many scRNA-seq methods](https://github.com/Teichlab/scg_lib_structs) other than 10x Genomics. Apparently, you cannot use Cell Ranger for all of them. Well ... actually you can if you modify your FASTQ files, check [UniverSC](https://www.nature.com/articles/s41467-022-34681-z) if you have not come across it.

Apparently, if we want an "apples-to-apples" comparison, a uniform preprocessing pipeline to convert the raw FASTQ reads to cell-by-gene count matrices is absolutely required. The idea here is to use **flexible**, **light-weighted** and **open-source** softwares to build such preprocessing procedures. With the development of the computational methods towards this purpose, we actually have many choices, such as [STARsolo](https://www.biorxiv.org/content/10.1101/2021.05.05.442755v1), [kallisto bustools](https://www.nature.com/articles/s41587-021-00870-2), [simpleaf/alevin-fry](https://www.nature.com/articles/s41592-022-01408-3) and [zUMIs](https://academic.oup.com/gigascience/article/7/6/giy059/5005022).

## Some Solutions

In order to build the preprocessing pipelines, we need to know the library structures of each single-cell method. That is, the identity of each base in a final library. Is the base in adapters, cDNA, cell barcodes *etc*.? With the ever-growing complexity of single-cell methods, figuring out the library structures is much more difficult than one might expect. How difficult is it to figure out the details from the method section of the paper? Well ... it turns out to be very difficult, which we will demonstrate. That was the reason I created the [scg_lib_structs GitHub](https://github.com/Teichlab/scg_lib_structs) repository. The HTML files there help visualise how the final library is generated in each method. They are good to read by eyes, but they are not written in a structured way such that you can parse the files systematically.

Recently, thanks to [Sina Booeshaghi](https://twitter.com/sinabooeshaghi) and [Lior Pachter](https://twitter.com/lpachter), we now have a machine-readable format: [*seqspec*](https://www.biorxiv.org/content/10.1101/2023.03.17.533215v1). In *seqspec*, complicated libraries are treated as **meta regions** which are formed by joining **atomic regions**, the basic components representing DNA fragments. Using *seqspec*, the final library structure of a method is **standardised** in a **machine-readable** way. In the future when new methods are developed, the developers can submit the raw FASTQ files together with a *seqspec* specification. Then the user would immediately understand the library structure and the identity of each base in the library. Once the library structure is clear, the commands needed for the preprocessing pipeline are straightforward to write. All of those can be done systematically using scripts since *seqspec* is machine-readable, . 

## An Example

Finally, one extra complexity for single-cell methods is that the field is developing so fast! How fast? It is so fast that when a new method is published, it is already outdated. The recent [PIP-seq](https://www.nature.com/articles/s41587-023-01685-z) method is a perfect example. There are three main ways to achieve high-throughput single-cell profiling:

1. Droplets (**Drop-seq**, **inDrop**, **10x Genomics**, *etc*.)
2. Nanowells (**Seq-Well**, **BD Rhapsody**, *etc*.)
3. Combinatorial indexing (**sci-RNA-seq**, **SPLiT-seq**, *etc*.)

Enters **PIP-seq**. They say: you can do that by simple vortexing. Well ... technically you still need microfluidics device to generate barcoded beads, but they have a company ([FluentBio](https://www.fluentbio.com)) to do that for you. In the end, if you use their products, all you need to do is really indeed just vortexing. **This is really amazing**. By the way, the commercialised version of PIP-seq by FluentBio is named PIPseq$^\textmd{TM}$.

If you really dig into the data from the PIP-seq paper, you will realise that there are three different versions of the library, representing different developmental stages of the method (I assume ...). Now the company announce there is a [fourth version](https://www.fluentbio.com/fluent-biosciences-announces-pipseq-v4-0-chemistry-with-significant-performance-enhancements-and-the-highest-cell-capture-rate-in-the-single-cell-market-at-the-2023-agbt-conference/), although no example data available at the time of writing.

Here, I think it is a good opportunity for me to document how I figured out and guessed the library structures of those three different versions of PIP-seq. I will post them separately in the next three posts, from the easiest (V3) to the hardest (V1). It should provide some rough ideas to people with less experience about how one should analyse the raw reads and what to look for in the FASTQ files.
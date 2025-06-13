---
title: "scg_lib_strucuts"
date: 2022-03-04T15:07:53+08:00
draft: true
type: post
---

document the procedures to re-generate blacklist

```
conda create -n encodebl
conda activate encodebl
conda install -c bioconda encode-blacklist umap
```

other things that may helpful:

remove input peaks
repeats: RNA, DNA LTR
bowtie, bwa vs hisat2
blacklist v1 vs v2
will t2t reference help?
---
title: "Hugo Syntax Highlight Bash"
date: 2024-01-08T09:24:53+08:00
draft: true
tags: ['writing', 'hugo', 'syntax highlighting', 'code block', 'bash']
type: post
---

{{< highlight console >}}
# some code
echo "Hello World"
$ cat tmp.txt
$ head -n 100 tmp.txt
$ tail -n 100 tmp.txt

# some awk code
tail -n +2 refseq_genes_hg38.txt | awk '
BEGIN{OFS="\t"}{
if($4=="+") {print $3,$5,$5+1,$2 "_" $13,".",$4}
 else {print $3,$6-1,$6,$2 "_" $13,".",$4}
 }' > refseq_hg38_UCSC_TSS.bed
{{< /highlight >}}

```console
$ echo "Hello World"
$ cat tmp.txt
$ head -n 100 tmp.txt
$ tail -n 100 tmp.txt
```
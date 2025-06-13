---
title: Set Up Public Links For UCSC Genome Browser Visualisation At The WTSI
date: 2018-07-02T00:00:00+08:00
tags: ['bioinformatics', 'unix', 'ucsc genome browser']
type: post
---

The best way of visualising sequencing reads pileup/coverage data (very useful for ChIP-seq, DNase-seq & ATAC-seq) is to get the pileup/coverage files in __[bigWig format](https://genome.ucsc.edu/goldenpath/help/bigWig.html)__, and visualise the signal as custom tracks using __[UCSC genome browser](https://genome.ucsc.edu/goldenpath/help/customTrack.html)__ - the best genome browser, period! The bigWig files are often large, especially when you have high coverage experiments. Therefore, you need a place to host your files, and they need to be in a place where UCSC genome browser can reach.

At Sanger, it is not so obvious how you can do it, but it is not difficult. The information is kind of hidden, and you need to find the right person to ask. At this moment of writing, there are two eay ways of achieving this goal.

## Method 1: use web-bfint:/data

There is a host called `web-bfint`. You can simply access the host by `ssh your_sanger_id@web-bfint` and transfer files from/to it by `scp file_name your_sanger_id@web-bfint` as long as you are inside Sanger network. When you login to `web-bfint`, you will see a welcome message:

```
Welcome to Ubuntu 16.04 (xenial) (4.4.0-87-generic x86_64) 

* Managed by WTSI Ansible. 
  See the documentation here:
  https://ssg-confluence.internal.sanger.ac.uk/x/_YbRAw

Last login: Thu Jun  7 18:38:55 2018 from *.*.*.*
```

Once you are in `web-bfint`, there are two places that will be useful to you:

```bash
# for short term deposition of files for collaborators and research
# data will be removed after a while
#(I think it is every six months, not entirely sure though)

/data/scratch/project

# for long term files associated with formal journal publications or archival

/data/production
```

You create a ticket via ServiceDesk, asking them to create a directory there for your team under either `/data/scratch/project` or `/data/production` or both. Check with the IT person in your team, maybe you already have one there. For example, our team directory are `/data/scratch/project/team205` and `/data/production/teichmann`. The public address associated with those two directories are `ftp://ngs.sanger.ac.uk/scratch/project/team205` and `ftp://ngs.sanger.ac.uk/production/teichmann`, respectively. Anybody can access those places via those addresses anywhere, even outside Sanger network. You and your team members can create subdirectories for youselves.

For example, I have a bigWig file called `the_best_chip_seq_ever.bw` somewhere on Sanger farm, and I want to visualise it via UCSC genome browser. First, I need to transfer that file to my team directory on `web-bfint`. I locate the file on farm, and do:

```bash
scp the_best_chip_seq_ever.bw xc1@web-bfint:/data/production/teichmann/xi/
```

After the transfer, if I enter this address in my Chrome/Firefox `ftp://ngs.sanger.ac.uk/production/teichmann/xi`, I should be able to see the file there. Note: files are synced hourly, so you might exprience a delay between getting files to the directory and being able to see the files via the ftp address.

To visualise the file, I can just go to UCSC genome browser to add a custom track by (just an example, the following text should be in one single line):

```
track type=bigWig name=chip_seq description=the_best_chip_seq_ever_pileup
bigDataUrl=ftp://ngs.sanger.ac.uk/production/teichmann/xi/the_best_chip_seq_ever.bw
visibility=full maxHeightPixels=10:50:128 viewLimits=0:20 autoScale=OFF color=0,0,255
```

That's it.

## Method 2: use S3 storage

The ftp method is easy, but it has a problem. Sometimes, especially when you have large amount of data, the UCSC genome browser shows an error: `ftp server error on cmd=[PASS x@genome.ucsc.edu]`. You have to keep refreshing the page several times, which is annoying.

Another way is to use __[S3 storage](https://aws.amazon.com/s3/)__ provided under Sanger Flexible Computing Environment. Again, create a ticket via ServiceDesk, saying that you want to use S3 storage. You can request a maximum of 100 GB. 100 GB is quite small, but it can still host quite a large amount of bigWig files.

After they created an S3 account for you, you will have a `.s3cfg` file in your home directory on Sanger farm, and the content will look like this (I ommited security part):

```bash
cat ~/.s3cfg 

[default]
encrypt = False
host_base = cog.sanger.ac.uk
host_bucket = %(bucket)s.cog.sanger.ac.uk
progress_meter = True
use_https = True
access_key = *************
secret_key = *****************
```

You will need the tool `s3cmd`, which is already installed on Sanger farm. S3 storage is so-called object storage. It does not have a hierarchical structure like the file system on unix. Instead, you create buckets, and you put your files/objects in the bucket you created. The name of the bucket must be unique within the entire institute, so it is better to name the bucket related to your Sanger user name.

For example, I want to create a bucket named `xc1_mSp_scATAC` to put all my scATAC-seq bigWig files there, so, on Sanger farm, I do:

```bash
s3cmd mb s3://xc1_mSp_scATAC
```

Then, I have a bigWig file called `the_best_atac_seq_ever.bw` somewhere on the farm. To visualise it, I first need to put it in my s3 bucket that I just created. This can be achieved by locating the bigWig file on the farm and doing:

```bash
s3cmd put the_best_atac_seq_ever.bw s3://xc1_mSp_scATAC
```

By default, the file/object in the bucket is private. Then, I need to make the file public so that UCSC genome browser can reach it. This can be done by:

```bash
s3cmd setacl --acl-public s3://xc1_mSp_scATAC/the_best_atac_seq_ever.bw
```

Now, it is done. The public address for the bigWig file is `https://cog.sanger.ac.uk/xc1_mSp_scATAC/the_best_atac_seq_ever.bw` (now you see why the bucket name must be unique across the entire institute, right?).

To visualise the file, I can just go to UCSC genome browser to add a custom track by (similar example as the previous one, and again, the following text should be in one single line):

```
track type=bigWig name=atac_seq description=the_best_atac_seq_ever_pileup
bigDataUrl=https://cog.sanger.ac.uk/xc1_mSp_scATAC/the_best_atac_seq_ever.bw
visibility=full maxHeightPixels=10:50:128 viewLimits=0:20 autoScale=OFF color=0,0,255
```

For more `s3cmd` usage, such as listing buckets/files, creating and deleting buckets/files, you can simply do `s3cmd --help`.

Enjoy the visualisation.

![](https://raw.githubusercontent.com/dbrg77/plate_scATAC-seq/master/figures/ucsc_example_cxcr5_locus.jpg)

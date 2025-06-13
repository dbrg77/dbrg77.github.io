---
title: Use Aliyun OSS For UCSC Visualisation In China Mainland
date: 2019-05-02T00:00:00+08:00
tags: ['bioinformatics', 'unix', 'ucsc genome browser', 'aliyun', 'ossutils']
type: post
---

You always need to repeat some basic tasks that you did in the past when you relocate to a new place. For some of them, you need to find alternative ways of achieving the same goals. For example, finding a way of visualising __[bigWig](https://genome.ucsc.edu/goldenpath/help/bigWig.html)__ signal files. This is very important for the assessment of the data quality. I hate to say it, but it is often true that the most efficient way of telling whether an experiment works or not is the least scientific way, which is by visual inspection. Looking at bigWig signals is relatively easy in [my previous institute](https://randomstate.net/2018-07-02-setting-up-public-links-for-ucsc-visualisation/), but it is not so straigtforward for a small lab that does not have good computational support. Things become even more annoying and complicated when your lab is in China mainland for obvious reasons.

## The problem.

The bigWig files we got nowadays are often large (>100 MB), so it is not possible to physically upload them to the UCSC genome browser. You need a server to host the files. You put your bigWig files on the server and allow them to be accessed publicly so that UCSC genome browser can reach them. Of course, you can use sevices like [Dropbox](https://www.dropbox.com/) or [Amazon S3](https://aws.amazon.com/s3/), but they are either blocked in China mainland or extremely slow. Try upload a bigWig file generated from only 1 million sequencing reads from your univeristy network. It takes ages here. Therefore, we need to solve both the server problem and the internet speed problem.

## The Goal.

1. Find a server that is publicly reachable to host your files.
2. Speed needs to be fast: the connection between your local machine and the server, and the connection between the server and the UCSC genome browser.
3. Affordable.

## The Solution.

Having looked around for a few things, I think I find a solution. This may not be THE solution, because I haven't tried hard enough. This solution is to use [Aliyun Object Storage Service](https://www.aliyun.com/), which is similar to Amazon S3, and it really solves all the problems mentioned above. The price is pretty affordable. For 100GB of data, you only pay 12 RMB per month. With 100GB of space, you can store quite a lot of experiments. The price is nothing comparing to the reagents you spent on your experiments, and you don't need to worry about the maintaince of the sever etc.

![](/images/000/oss_price.png)

To use it, purchase the service, and login to the [OSS console](https://oss.console.aliyun.com). Now at the top right corner, click the `Access Key` button:

![](/images/000/oss_console.png)

Then a warning will pop up, but you just click "Continue to use Access Key". Then you can get your `Access Key ID` and your `Access Key Secret`:

![](/images/000/oss_accesskey.png)

Now in your local machine, open up your `Terminal` and download the right binary from [this page](https://help.aliyun.com/document_detail/50452.html?spm=a2c4g.11186623.6.1382.fe817e90clvugi). Start configure your OSS storage by type:

```
ossutil64 config
```

Just input the information required, and here is mine:

```
For the following settings, carriage return means skip the configuration. Please try "help config" to see the meaning of the settings
Please enter endpoint:https://oss-cn-shenzhen.aliyuncs.com
Please enter accessKeyID:your_access_key_id
Please enter accessKeySecret:your_access_key_secret
Please enter stsToken:
```

The `stsToken` is optional, so just press <kbd>Enter</kbd>. You have the following options for endpoints:

```
https://oss-cn-hangzhou.aliyuncs.com
https://oss-cn-shanghai.aliyuncs.com
https://oss-cn-qingdao.aliyuncs.com
https://oss-cn-beijing.aliyuncs.com
https://oss-cn-zhangjiakou.aliyuncs.com
https://oss-cn-huhehaote.aliyuncs.com
https://oss-cn-shenzhen.aliyuncs.com
https://oss-cn-hongkong.aliyuncs.com
...

and a few more in other countries.
```

Okay, just like Amazon S3, you need to create a bucket and put your files into the bucket. For example, I created a bucket called `myatacbucket` by (Note: only lowercase letters, numbers and `-` are allowed to name a bucket):

```bash
ossutil64 mb oss://myatacbucket --acl=public-read --storage-class Standard
```

This creates a bucket called `myatacbucket` that can be read by public. Okay, now locate the bigWig file in your local machine, and transfer it to the bucket:

```bash
ossutil64 cp my_bigwig.bw oss://myatacbucket
```

Now, to get an URL so that UCSC can reach it. You go to your OSS console, and do `your bucket name --> File Management --> Tick your files --> Export URL`, like this:

![](/images/000/oss_url.png)

So, using the above example, my URL will be:

```
https://myatacbucket.oss-cn-shenzhen.aliyuncs.com/my_bigwig.bw
```

Go to UCSC (recommend the Asia mirror if you are in China: [https://genome-asia.ucsc.edu](https://genome-asia.ucsc.edu)), and put your URL there like usual (the following text should be in one single line):

```
track type=bigWig name=atac_seq description=the_best_atac_seq_ever_pileup
bigDataUrl=https://myatacbucket.oss-cn-shenzhen.aliyuncs.com/my_bigwig.bw
visibility=full maxHeightPixels=10:50:128 viewLimits=0:20 autoScale=OFF color=0,0,255
```

Now it is done, and the speed is quite nice. You can find more information about ossutils from their [documentation](https://help.aliyun.com/document_detail/50452.html?spm=a2c4g.11186623.6.1382.1e0a1594MxR9iZ).

![](/images/000/ucsc_vis.png)

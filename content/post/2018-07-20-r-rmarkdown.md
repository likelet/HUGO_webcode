---
title: "Blogs_tips from old site"
author: "Frida Gomam"
date: 2018-07-08T21:13:14-05:00
categories: ["R"]
tags: ["R Markdown", "setting", "regression"]
---

# Blogs_tips

## Table of Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

  - [Subset bamfile with chromosome names and convert into paired fastq](#subset-bamfile-with-chromosome-names-and-convert-into-paired-fastq)
  - [Cluster management](#cluster-management)
  - [R code for ploting nomograph from competing risk survival analysis model](#r-code-for-ploting-nomograph-from-competing-risk-survival-analysis-model)
  - [Setting docker download mirror site](#setting-docker-download-mirror-site)
  - [Install bioconductor R package using VPS](#install-bioconductor-r-package-using-vps)
  - [Install bioconductor R package using mirror at UTSC](#install-bioconductor-r-package-using-mirror-at-utsc)
  - [Tips for using Tianhe-2 super computer](#tips-for-using-tianhe-super-computer)



## Subset bamfile with chromosome names and convert into paired fastq  
* software required: **[sambamba](https://github.com/lomereiter/sambamba)** and **[bam2fastx](https://github.com/infphilo/tophat)** from tophat binary distribution.<br>

 > sambamba usages should refer to https://github.com/lomereiter/sambamba/wiki/%5Bsambamba-view%5D-Filter-expression-syntax#basic-conditions-for-fields

```shell 
#using star output bamfile as example 
#!/bin/sh
bamin=$1
#extract reads aligned to chr2
sambamba view -F "ref_id==1" -f bam $bamin -o ${bamin%%Aligned.sortedByCoord.out.bam}_chr2.bam
#sort reads by names if not presorted by software
sambamba sort -n ${bamin%%Aligned.sortedByCoord.out.bam}_chr2.bam -o ${bamin%%Aligned.sortedByCoord.out.bam}_chr2.sort.bam
#bam2fastq
bam2fastx -PANQ -o ${bamin%%Aligned.sortedByCoord.out.bam}_chr2.fq.gz ${bamin%%Aligned.sortedByCoord.out.bam}_chr2.sort.bam

```
**PS**: the numbers specified in `ref_id` means the ref order list in header from bamfle, which can be checked by 
`samtools view -H your.bam` if samtools was installed. 


## Cluster management 
* 1. shudown system 
Shut down computational node 
```shell
#!/bin/sh
for i in `seq 1 3`
do
 ssh cu0$i "hostname;init 0"
done
```
umount storage 
```shell
umount /home
```
shutdown login node 
```shell
poweroff
```
## R code for ploting nomograph from competing risk survival analysis model 
```R
library(cmprsk)
library(rms)
### add path 
setwd("C:\\Users\\hh\\Desktop\\nomo")
rt<-read.csv("Stomach.csv")
rt
View(rt)
attach(rt) 
#change variable names

cov<-cbind(sexC, Age, AJCC_T,AJCC_N,AJCC_M,Surgery)
for (i in 1:6)
{
  cov[,i]<-factor(cov[,i])
}
status<-factor(status)
z <- crr(time,status,cov)
z.p <- predict(z,cov)
n=60#suppose I want to predict the probability of event at time 60(an order)
df<-data.frame(y=z.p[n,-1],cov)
ddist <- datadist(df)  
options(datadist='ddist') 
lmod<-ols(y~(sexC)+(Age)+(AJCC_T)+(AJCC_N)+(AJCC_M)+(Surgery),data=df)#
nom<-nomogram(lmod)
plot(nom,lplabel=paste("prob. of incidence T",round(z.p[n,1],2),sep="="))
```
## Setting docker download mirror site 
Sometimes you may find that it's extrimely painfull to pull docker image from docker.io in china. So this tip can help you to set a mirror site locally in your docker pull command.  
* 1. First, find the file `/etc/docker/daemon.json` and modify it with root authority.
```{javascript}
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```
* 2. Secondly, restart your docker service. 
## Install bioconductor R package using VPS.   

    proxychains4 Rscript -e 'source("http://bioconductor.org/biocLite.R"); biocLite("BSgenome")'

## install bioconductor R package using mirror at UTSC. 

    source("http://bioconductor.org/biocLite.R")
    options(BioC_mirror="http://mirrors.ustc.edu.cn/bioc/")
    biocLite("your package")

## Tips for using Tianhe super computer  

* 1. Logging in the data transfer server from rj account  

      ssh -p 5566 ln42  
      ssh tn1-ib0
## update R packages   

    # set mirror in local china 
    options(BioC_mirror="http://mirrors.tuna.tsinghua.edu.cn/bioconductor")
    update.packages(checkBuilt=TRUE, ask=FALSE)
    

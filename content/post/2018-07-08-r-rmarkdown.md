---
title: "Tabix your bed file to parse your bed quickly "
author: "Qi Zhao"
date: 2018-07-20T21:13:14-05:00
categories: ["install"]
tags: ["bed", "processing", "data format"]
---

## Tabix your bed file to parse your bed quickly 

It is the first time that I realize that the `bed`, `gtf`, `gff`and `vcf` format file  can be indexed to improve their parsing efficiency. 
The story is come from the user experience for strelka2. The strelka2 was publihsed recently and i try to use it for claimed improvement on both accuracy and performance. I used the strelka1 before and found the false positive pos by such software(see this [issue](https://github.com/Illumina/strelka/issues/12) ). Although the problem is not fixed yet, I still want to have a try on my current data. Hope it will be fixed soon.   

 
### install Tabix 

The detailed description of tabix can be found [here](http://www.htslib.org/doc/tabix.html).  

To install, i empoyed `conda` package management system. Simplely by this command  

		sudo conda install -i bioconda tabix 
       
### Sort your bed  

Tabix required your bed sorted, which can be fulfiled by this  

		sort -k 1,1 -k 2,2n -k 3,3n your.bed | bgzip -c > your.bed.gz
        
### tabix it 

Index your bed  

		tabix -pbed your.bed.gz 
        
then you can find a `your.bed.gz.tbi` file in your currenty folder. 

### run strelka2 for somantic variants calling 

```shell
        #!/bin/sh
        # read args
        tumor=$1
        normal=$2


        # set path 
        genome=/data/database/human/hg38/GDCref/GRCh38.d1.vd1.fa
        targetBed=/backup/PDX/bedfile/Exome-Agilent_V6_hg38_chr.bed.gz
        tumorbam=${tumor}-WES_sort_dedup_realigned_recal.bam
        normalbam=${normal}-WES_sort_dedup_realigned_recal.bam

        # creat directories 
        if [ ! -d ${tumor}_strelka  ];then
          mkdir ${tumor}_strelka
        else
          rm -rf ${tumor}_strelka
          echo ${tumor}_strelka exist
          echo overwrite anyway
          mkdir ${tumor}_strelka
        fi

        # run configuration 

        ${STRELKA_INSTALL_PATH}/bin/configureStrelkaSomaticWorkflow.py \
            --normalBam $normalbam \
            --tumorBam $tumorbam \
            --ref $genome \
            --callRegions $targetBed \
            --runDir ${tumor}_strelka

        ${tumor}_strelka/runWorkflow.py -m local -j 20

```
    

## ChIPseqSpikeInFree

A Spike-in Free ChIP-Seq Normalization Approach for Detecting Global Changes in Histone Modifications

## Background

Traditional reads per million (RPM) normalization method is inappropriate for the evaluation of ChIP-seq data when the treatment or mutation has the global effect. Changes in global levels of histone modifications can be detected by using exogenous reference spike-in controls. However, most of the ChIP-seq studies have overlooked the normalization problem that have to be corrected with spike-in. A method that retrospectively renormalize data sets without spike-in is lacking. 

We observed that some highly enriched regions were retained despite global changes by oncogenic mutations or drug treatment and that the proportion of reads within these regions was inversely associated with total histone mark levels. Therefore, we developped `ChIPseqSpikeInFree`, a novel ChIP-seq normalization method to effectively determine scaling factors for samples across various conditions and treatments, which does not rely on exogenous spike-in chromatin or peak detection to reveal global changes in histone modification occupancy. This method is capable of revealing the similar magnitude of global changes as the spike-in method.

In summary, `ChIPseqSpikeInFree` can estimate scaling factors for ChIP-seq samples without exogenous spike-in or without input. When ChIP-seq is done with spike-in protocol but high variation of Spike-In reads between samples are observed,  ChIPseqSpikeInFree can help you determine a more reliable scaling factor than ChIP-Rx method. It's not recommended to run ChIPseqSpikeInFree blindly without any biological evidences like Western Blotting to prove the global change at protein level between your control and treatment samples. 


## App on DNAnexus Cloud platform - No installation, click and run.
To use the tool, you will need to create a DNAnexus account at https://platform.dnanexus.com/register?client_id=sjcloudplatform.   After logging in DNAnexus,  you can create a project , upload your data to your project folder and choose the ChIPseqSpikeInFree app (Tools --> library --> search ChIPseqSpikeInFree) to run.  Or you can run ChIPseqSpikeInFree [v1.2.3] at https://platform.dnanexus.com/app/ChIPseqSpikeInFree and get results in an hour.


## Prerequisites

`ChIPseqSpikeInFree` depends on `Rsamtools`, `GenomicRanges`, and `GenomicAlignments` to count reads from bam files.

To install these packages, start `R` (version "3.5")or above, enter:
```R
> if (!requireNamespace("BiocManager", quietly = TRUE))
>     install.packages("BiocManager")
> BiocManager::install("Rsamtools")
> BiocManager::install("GenomicRanges")
> BiocManager::install("GenomicAlignments")
```

## Installation

Using `R`, enter:

```R
# Install this package from GitHub
> install.packages("devtools")
> library(devtools)
> install_github("stjude/ChIPseqSpikeInFree")
> packageVersion('ChIPseqSpikeInFree')
#[1] '1.2.4'
```

Or using command lines

```bash
$ git clone https://github.com/stjude/ChIPseqSpikeInFree.git
$ R CMD build ChIPseqSpikeInFree
$ R CMD INSTALL ChIPseqSpikeInFree_1.2.4.tar.gz

```

### Usage

A simple workflow in `R` environment.

##### 0. Load package

```R
> library("ChIPseqSpikeInFree")
```

##### 1. Generate `sample_meta.txt` (**tab-delimited txt file with header line**) as follows

Save as `/your/path/sample_meta.txt` ([Example](docs/sample_meta.txt))

| ID                        | ANTIBODY | GROUP |
| ------------------------- | -------- | ----- |
| WT-H3K27me3.rmdupq1.bam   | H3K27me3 | WT    |
| K27M-H3K27me3.rmdupq1.bam | H3K27me3 | K27M  |
| WT-INPUT.rmdupq1.bam      | INPUT    | WT    |
| K27M-INPUT.rmdupq1.bam    | INPUT    | K27M  |

**Note:** INPUT samples are not required at all for a valid metadata file. 

```R
> metaFile <- "/your/path/sample_meta.txt"
```

##### 2. Assign bam file names to a vector

```R
> bams <- c("WT-H3K27me3.rmdupq1.bam","K27M-H3K27me3.rmdupq1.bam")
```

##### 3. Run `ChIPseqSpikeInFree` pipeline (when your bam files correspond to the human reference hg19)

```R
> ChIPseqSpikeInFree(bamFiles = bams, chromFile = "hg19", metaFile = metaFile, prefix = "test")
```

##### 4. Run `ChIPseqSpikeInFree` pipeline with custom settings for ChIP-seq with unideal enrichment or many very broad enriched regions like H3K9me3

```R
> ChIPseqSpikeInFree(bamFiles = bams, chromFile = "hg19", metaFile = metaFile, prefix = "test", cutoff_QC = 1, maxLastTurn=0.97)
```

### Input

In the simple usage scenario, the user should have ChIP-seq bam files ready. Sample information can be specified in a metadata file (`metaFile`) and the user should choose a correct reference genome corresponding to the bams.

##### 1. bamFiles: a vector of bam filenames

User should follow ChIP-seq guidelines suggested by `ENCODE consortium(Landt, et al., 2012)` and check the data quality first. Some steps to check quality and prepare ChIP-seq bam files for `ChIPseqSpikeInFree` normalization:

* run SPP tool (Marinov, et al., 2014) to do ChIP-seq data QC and use samples with **Qtag >= 1**
* **remove spike-in reads** from your bam files
* only use **good-quality or uniquely-mapped reads** your bam files
	```bash
	java -jar picard.jar MarkDuplicates \
	      I=input.bam \
	      O=rmdup.bam \
	      M=rmdup_metrics.txt\
	      CREATE_INDEX=true \
              ASSUME_SORTED=false \
              REMOVE_DUPLICATES=true"
	samtools view -hb -q 1  rmdup.bam > rmdupq1.bam
	```
* bam files must **contain a header section** and an alignment section
	```bash
	samtools view -H rmdupq1.bam
	```

##### 2. chromFile: chromosome sizes of reference genome.
`hg19`, `mm9`, `mm10`, `hg38` are included in the package.

For other genomes, you can either:

* Use `fetchChromSizes` to get it from UCSC, but not all genomes are available.

    `http://hgdownload.soe.ucsc.edu/goldenPath/${DB}/bigZips/${DB}.chrom.sizes`  
    Replace `${DB}` with reference genome

* Generate directly from `fasta` file (Linux)
    ```bash
    $ samtools faidx genome.fa
    $ cut -f1,2 genome.fa.fai > genome.chrom.sizes
    ```

##### 3. metaFile: A tab-delimited text file **having three columns with a header line: `ID`, `ANTIBODY`, and `GROUP`**.

* `ID` is the bam file name of ChIP-seq sample that will be included for analysis.
* `ANTIBODY` represents antibody used for ChIP.
* `GROUP` describes the biological treatment or condition of this sample.

### Output

After you successfully run following `ChIPseqSpikeInFree` pipeline:
```R
> ChIPseqSpikeInFree(bamFiles = bams, chromFile = "hg19", metaFile = metaFile, prefix = "test")
```

Output will include: (in case that you set `prefix ="test"`)
1. `test_SF.txt` - text result (see [Interpretation section](#Interpretation-of-scaling-factor-table) or [text file](docs/test_SF.txt))
    * tab-delimited text format, a table of calculated scaling factors by pipeline
2. `test_distribution.pdf` - graphical result (see [Figure 1.A,B](#Graphical-results) or [PDF file](docs/test_distribution.pdf))
    * view of proportion of reads below the given CPMW based on `test_parsedMatrix.txt`
3. `test_boxplot.pdf` - graphical result  (see [Figure 1.C](#Graphical-results) or [PDF file](docs/test_boxplot.pdf))
    * view of scaling factors as boxplot based on `test_SF.txt`
4. `test_rawCounts.txt` - intermediate file
    * tab-delimited text format, a table of raw read counts for each 1kb bin across genome
5. `test_parsedMatrix.txt` - intermediate file
    * tab-delimited text format, a table of proportion of reads below given cutoffs (CPMW)
### Interpretation of scaling factor table  

|    ID                     | GROUP | ANTIBODY | COLOR | QC                                             | SF    |	TURNS                  |
| ------------------------- | ----- | -------- | ----- | ---------------------------------------------- | ----- | ------------------------ |
|WT-H3K27me3.rmdupq1.bam    | WT    | H3K27me3 | grey  | pass                                           | 1     |  0.25,0.3817,6.1,0.9523  |
|K27M-H3K27me3.rmdupq1.bam  | K27M  | H3K27me3 | green | pass                                           | 5.47  |  0.2,0.3429,34.7,0.9585  |
|WT-INPUT.rmdupq1.bam       | WT    | INPUT    | black | fail: complete loss, input or poor enrichment  | NA    |  0.2,0.1860,0.7,0.9396   |
|K27M-INPUT.rmdupq1.bam     | K27M  | INPUT    | grey  | fail: complete loss, input or poor enrichment  | NA    |  0.15,0.0675,0.35,0.8476 |

- **COLOR**: defines color for each sample in cumulative distribution curve `*_distribution.pdf`
- **QC**: quality control testing. QC failure indicates poor or no enrichment. 
- **SF**: scaling factor. Only sample that passes QC will be given a SF and **NA** indicates sample with poor enrichment. Input sample is not required for SF calculation. Larger scaling factor indicates a lower level of total histone mark and vice versa. For example, H3K27me3 is globally decreased in H3.3 K27M cells compared to WT  (**SF, 5.47 vs 1**). 
- **TURNS**: the coordinates of two points [Xa, Ya, Xb, Yb] detected in cumulative distribution plot (proportion of reads below CPMW cutoffs) for slope-based SF calculation. 

### How is scaling factor calculated?  
Given a set of n samples immunoprecipitated with the same antibody, we chose one sample as a reference βr. We derived the scaling factor (S) for any other sample i as Si=βr/βi where βi is the slope of sample i.  Since the scaling factor of one specific sample is relative to the reference sample, the value could be changed based on set of samples included in the analysis. The recommended comparison would be of individual samples between different GROUPs but with same ANTIBODY. 


### Graphical results
<img align="center" width="900" height="400" src="docs/H3K27me3_Fig1.jpg">

## How to use ChIPSeqSpikeIn scaling factor? 

We can use SF to adjust original library size for differential analysis and generating bigwig files.    

1. for differential analysis with EdgeR
```R
...
dat <- read.table("sample_SF.txt", sep="\t",header=TRUE,fill=TRUE,stringsAsFactors = FALSE, quote="",check.names=F)
SF <- dat$SF
dge <- DGEList(counts = counts, group = GROUP, norm.factors = SF)
...
```
2. for differential analysis with DESeq2
```R
...
dat <- read.table("sample_SF.txt", sep="\t",header=TRUE,fill=TRUE,stringsAsFactors = FALSE, quote="",check.names=F)
SF <- dat$SF
countData <- matrix(1:100,ncol=4)
condition <- factor(c("A","A","B","B"))
dds <- DESeqDataSetFromMatrix(countData, DataFrame(condition), ~ condition)
dds <- estimateSizeFactors(dds)
coldata<- colData(dds)
coldata$sizeFactor <- coldata$sizeFactor *SF
colData(dds) <- coldata
dds <- estimateDispersions(dds)
dds <- nbinomWaldTest(dds)
...
```

3.  pseudo-code for generation of bigwig files from a bed file
```bash
libSize=`cat sample1.bed|wc -l`
scale=15000000/($libSize*$SF)
genomeCoverageBed -bg -scale $scale -i sample1.bed  -g mm9.chromSizes > sample1.bedGraph
bedGraphToBigWig sample1.bedGraph mm9.chromSizes sample1.bw
```

## What happens if 100% complete loss is expected?

In some scenario, a histone mark can be absent in a cell type at a certain developmental stage or in a model that the histone writer gene has been knocked-out. In this case,  you will see a "NA" in SF column and  "fail: complete loss, input or poor enrichment" in QC column in the output file ${prefix}_SF.txt. It's unreasonable to calculate SF for this kind of samples alone.  However, if you restore the histone mark by drug treatment or ectopic over-xpression of  histone writer,  you can have a valid SF.  Now you may want to compare "NA" with non-"NA" SF for a histone mark to show a global change in bigwig file or differential analysis.  Here is the pseudocode  to transform SF:
 ```R
dat<-  read.table("sample_SF.txt", sep="\t",header=TRUE,fill=TRUE,stringsAsFactors = FALSE, quote="",check.names=F)
SF <- dat$SF
SF[is.na(SF)] <- 1
SF[!is.na(SF)]  <- 1/SF[!is.na(SF)]
dat$SF <- SF
write.table(dat, "sample_SF_completeLoss.txt", sep="\t",quote=F,row.names=F, col.names=T)
```

## Notes

This repository contains the following:
- source code
- documentation [[PDF file](docs/ChIPseqSpikeInFree_1.2.4.pdf)]
- chromFile of human and mouse reference genome (`hg19`, `mm9`, `mm10`, and `hg38`)
- an example of `sample_meta.txt`

Users will still need to source and provide the following:
- bam files
- `sample_meta.txt`
- chromFile except `hg19`, `mm9`, `mm10`, and `hg38`

## Maintainers

* [Hongjian Jin]

[Hongjian Jin]: https://github.com/hongjianjin

## How to cite
H Jin, LH Kasper, JD Larson, G Wu, SJ Baker, J Zhang, Y Fan*. (2019) ChIPseqSpikeInFree: A ChIP-seq normalization approach to reveal global changes in histone modifications without spike-in. Bioinformatics, https://doi.org/10.1093/bioinformatics/btz720

## Contact
ChIPseqSpikeInFree is developed by Drs. Hongjian Jin and Yiping Fan at the St Jude Children's Research Hospital. For collaborations or any other matters, please contact yiping.fanATstjude.org.

## R Session Info
```R
>sessionInfo()
R version 3.6.1 (2019-07-05)
Platform: x86_64-pc-linux-gnu (64-bit)
Running under: Red Hat Enterprise Linux

Matrix products: default
BLAS:   /home/R/install/3.6.1/lib64/R/lib/libRblas.so
LAPACK: /home/R/install/3.6.1/lib64/R/lib/libRlapack.so

locale:
[1] C

attached base packages:
[1] stats4    parallel  stats     graphics  grDevices utils     datasets
[8] methods   base

other attached packages:
 [1] ChIPseqSpikeInFree_1.2.4    GenomicAlignments_1.20.1
 [3] Rsamtools_2.0.3             Biostrings_2.52.0
 [5] XVector_0.24.0              SummarizedExperiment_1.14.1
 [7] DelayedArray_0.10.0         BiocParallel_1.18.1
 [9] matrixStats_0.55.0          Biobase_2.44.0
[11] GenomicRanges_1.36.1        GenomeInfoDb_1.20.0
[13] IRanges_2.18.3              S4Vectors_0.22.1
[15] BiocGenerics_0.30.0

loaded via a namespace (and not attached):
[1] lattice_0.20-38        bitops_1.0-6           grid_3.6.1
[4] zlibbioc_1.30.0        Matrix_1.2-17          tools_3.6.1
[7] RCurl_1.95-4.12        compiler_3.6.1         GenomeInfoDbData_1.2.1
```
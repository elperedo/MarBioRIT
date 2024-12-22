## Getting data from NCBI short read archive (SRA)

**In this example we are downloading is amplicon from NCBI.**


1. **Go to NCBI https://www.ncbi.nlm.nih.gov/ and select SRA in the database menu. Also, you can search in the project database.**

2. **You can search samples of interest by description, e.g. human microbiome, or by using an SRA sample number (usually these numbers will appear in the papers as data are usually required to be deposited in a public database before publication)**

3. example. Search in the search bar for 'salt marsh amplicon'. Select a sample of interest, e.g. 16SRNA-gene-saltmarshsediment

![](https://i.imgur.com/61LRPrN.png)

![](https://i.imgur.com/mFp1KeV.png)

    a. What is this sample from?
    b. What sequencing technology was used?
    c. When was it published?
    d. Does it belong to a bigger study?

4. **Open the Project accession in a separate tab.**

![](https://i.imgur.com/Dgr2pAO.png)

5. **Click on the SRA experiments link** (the number 53). A new tab will open. You can now send the results to the **run selector** (link in the center top). This will provide you access to each sample within the project and also to the metadata of the experiment.
Download the metadata and accession list files. these are text files, separated by commas. You can import the files into excel (search into how to import data, keywords for office Data import, text, tabulated commas). to avoid confusion add the project name to the file name

e.g. 

* SRR_Acc_List_PRJNA610907.txt
* SraRunTable_PRJNA610907.txt



![](https://i.imgur.com/JULPIaU.png)


6. **Click in run SRR11277344**. You will navigate to a new menu with lots of info about the sample.

However, you cannot process or analyze the full scope of the information contained in this file just by looking at it. To really analyze this data set, you will need to download it to your computer.

### Downloading data from NCBI

We are going to be downloading the data from the project PRJNA610907

[Documentation from NCBI](https://ncbi.github.io/sra-tools/fastq-dump.html)
On the terminal create a folder for this analysis

```
mkdir saltmarsh
cd saltmarsh
```

make a folder for the raw data

```
mkdir SRAdump_PRJNA610907
```
this will create a folder for you to download your data.
```
cd SRAdump_PRJNA610907
```
navigate to the folder.
we will use fastq-dump to download the data.

We have already checked the data characteristics. 

This is very important because we need to know how to process them afterward. This is a set of samples sequenced using [paired-end reads](https://www.illumina.com/science/technology/next-generation-sequencing/plan-experiments/paired-end-vs-single-read.html)
so we want to separate each read into separate files 1/ and 2/. To do so, we will use the split files flag. we will also skip any technical reads that might be present)

[SRR11277344](https://trace.ncbi.nlm.nih.gov/Traces/sra/?run=SRR11277344)


### Prefetch


We will download files from NCBI using `prefetch` so we will get the data in compressed .sra format. Then we will use `fastq-dump` to uncompress the files (note, fastq-dump can also be used to download files)

http://www.metagenomics.wiki/tools/short-read/ncbi-sra-file-format/prefetch 

https://edwards.sdsu.edu/research/fastq-dump/

https://ncbi.github.io/sra-tools/fastq-dump.html 



check that you are in the right directory using `pwd`

is it `~/saltmarsh/SRAdump_PRJNA610907`?

Now, let's download some files!
Type
```
prefetch -f yes SRR11277344
```
Use `ls` to check if the file is present. 

```
vdb-validate ~/saltmarsh/SRAdump_PRJNA610907/SRR11277344/SRR11277344.sra

```
You can use `**vdb-validate**` for checking the download
[documentation from NCBI](https://trace.ncbi.nlm.nih.gov/Traces/sra/sra.cgi?view=toolkit_doc&f=vdb-validate)

:::success
#[eperedo@class-02 SRR11277344$] vdb-validate ~/saltmarsh/SRAdump_PRJNA610907/SRR11277344/SRR11277344.sra

2020-10-08T16:45:43 vdb-validate.2.10.0 info: Database 'SRR11277344.sra' metadata: md5 ok

2020-10-08T16:45:43 vdb-validate.2.10.0 info: Table 'SEQUENCE' metadata: md5 ok

2020-10-08T16:45:43 vdb-validate.2.10.0 info: Column 'QUALITY': checksums ok

2020-10-08T16:45:43 vdb-validate.2.10.0 info: Column 'READ': checksums ok

2020-10-08T16:45:43 vdb-validate.2.10.0 info: Column 'READ_LEN': checksums ok

2020-10-08T16:45:43 vdb-validate.2.10.0 info: Column 'READ_START': checksums ok

2020-10-08T16:45:43 vdb-validate.2.10.0 info: Database '/users/eperedo/saltmarsh/SRAdump_PRJNA610907/SRR11277344/SRR11277344.sra' contains only unaligned reads

2020-10-08T16:45:43 vdb-validate.2.10.0 info: Database 'SRR11277344.sra' is consistent

:::

```
fastq-dump  \
    -W \
    --split-files \
    --read-filter pass \
    --gzip \
    --skip-technical \
    ~/saltmarsh/SRAdump_PRJNA610907/SRR11277344/SRR11277344.sra
```
in `fastq-dump`

* The flag -W will remove adapters if present. 
* --split-files will separate R1 and R2
* --gzip will compress the files 
* --read-filter pass removes low-quality data
* --skip-technical remove technical reads if present. 

:::info
Read 61166 spots for /users/eperedo/saltmarsh/SRAdump_PRJNA610907/SRR11277344/SRR11277344.sra
Written 61166 spots for /users/eperedo/saltmarsh/SRAdump_PRJNA610907/SRR11277344/SRR11277344.sra

:::



If you do `ls` now, you should see two files.

SRR11277344_pass_1.fastq.gz  SRR11277344_pass_2.fastq.gz

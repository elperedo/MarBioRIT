# Downloading Multiple files
[TOC]

## Getting your terminal ready

Navigate to the folder where you will run your analysis. 
Hint, you will need to use `cd` `ls` `pwd`


## LOOPS
Of course, if you were analyzing a relatively long set of samples, you would not be doing this one by one!

To avoid having to do this one by one we will create a **for loop**. [more here](https://www.tutorialspoint.com/unix/unix-shell-loops.htm)
All we need is a text file containing the list of files you want to process. Here we will cover an example, using the same dataset as before. In the terminal, get out of the current folder and make a new one.

do you remember the file SRR_Acc_List_PRJNA610907.txt you downloaded?
copy it to the folder. (you can drag and drop using FileZilla or you can create a file using the terminal). 

```
cat > SRR_Acc_List_PRJNA610907.txt
```
this creates the file, let's populate it. We will only use 29 samples of this dataset (the ones identified as PCR in the metadata file)

Enter your document's text. open the txt file in your computer, select all (crt +a) and copy (crt +c) now copy all the text in the terminal **(crt + shift +v)** note: copy and past require crt +shift. 
```
SRR11277344
SRR11277345
SRR11277346
SRR11277347
SRR11277348
```
#Press enter to insert and end line.exit using Ctrl + Z
and check!
```
ls -l SRR_Acc_List_PRJNA610907.txt
head SRR_Acc_List_PRJNA610907.txt

```

### Loop it!

Example with fastq-dump

Now let's run all the commands we just used in the loop.

Check that you are in the right directory
~/saltmarsh/SRAdump_PRJNA610907/

```
for line in `cat SRR_Acc_List_PRJNA610907.txt`; \
    do prefetch -f yes ${line}; \
    done
```
```
for line in `cat SRR_Acc_List_PRJNA610907.txt`; \
    do vdb-validate ${line}/${line}.sra; \
    done
```

could you write the rest?


...

Or you can check below. (click on the carrot for solutions) 


:::spoiler


```
#download-- 
for line in `cat SRR_Acc_List_PRJNA610907.txt`; \
    do prefetch -f yes ${line}; \
    done

#validate--
for line in `cat SRR_Acc_List_PRJNA610907.txt`; \
    do vdb-validate ${line}/${line}.sra; \
    done

#uncompress
for line in `cat SRR_Acc_List_PRJNA610907.txt`; \
    do fastq-dump \
    -W \
    --split-files \
    --read-filter pass \
    --gzip \
    --skip-technical \
   ${line}/${line}.sra; \
    done
```
:::





[TOC]

# Part I. Statistical Analysis
## Visualizing your results

## How to present a long p-value list of results?
Quite often, you might have a long list of results when testing for beta diversity significance. One way of summarizing p-values is by generating a color-coded matrix.

[Example file here](https://docs.google.com/spreadsheets/d/1MrPo37VjbgAknfJ54cXetKfgiRUcsjFVC172oq6WHMs/edit?usp=sharing)

![](https://i.imgur.com/Gl4p3kb.png)



## Identifying shared and exclusive taxa (Venn diagrams). 
You can use a Venn diagram to identify the presence of shared taxa in all your treatments of those that are only found in one condition. 

You probably want to reduce a bit the degree of detail of this analysis and just focus on the species level. For that, use the collapsed taxonomy table to compare taxonomy among different types of samples using Venn diagrams. If you want to compare treatments, you will have to identify which sample belongs to each treatment and add the counts of each sample. For an example see [here ](https://docs.google.com/spreadsheets/d/1PX9prHu8zoX-chfppxebgJqi5YzJ6U2m/edit?usp=sharing&ouid=117013565017338924493&rtpof=true&sd=true)

Take all of the taxa or ASV names of the samples that you want to analyze, and put them into any of these Venn Diagram tools online:

https://bioinfogp.cnb.csic.es/tools/venny/
http://bioinformatics.psb.ugent.be/webtools/Venn/ 
http://www.interactivenn.net/

![](https://i.imgur.com/XUb6OKY.png)



## Bubble plots
You might want to summarize the fate during the experiments of some bacterial groups of interest. As for many of the analyses you could use the ASV level or the collapsed taxonomy. Because we want to compare the abundance of each taxon, and this is a quantitative measure, you need to use a normalized table. 
We can work with the `table-filtered-no-mitochondria-no-chloroplast-rarefied.qza` file we generated while calculating beta diversity. 
If you are interested in the ASV level, you can use the feature table within the folder `ASV-table-filtered-no-mitochondria-no-chloroplast-rarefied`

You want to focus on a higher taxonomical rank, so we can easily generate a  collapsed taxonomy as we did before. 


```
qiime taxa collapse  --i-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza --i-taxonomy taxonomy.qza   --o-collapsed-table table-collapsed-filtered-no-mitochondria-no-chloroplast-rarefied-level3.qza --p-level 3
```
You can export the table into excel for additional analysis.  
```
qiime tools export   --input-path table-collapsed-filtered-no-mitochondria-no-chloroplast-rarefied-level3.qza  --output-path table-collapsed-filtered-no-mitochondria-no-chloroplast-rarefied-level3
```

Convert the exported table from biom format to tsv
```
biom convert -i table-collapsed-filtered-no-mitochondria-no-chloroplast-rarefied-level3/feature-table.biom -o table-collapsed-filtered-no-mitochondria-no-chloroplast-rarefied-level3/feature-table.tsv --to-tsv
```
You can sort the taxa and select those that are of interest. There is a few transformations of the data you have to do. You can follow the instructions in this [example file](https://docs.google.com/spreadsheets/d/1-NiKGtMQ4TiaIEWu3uTKF6NbbfQILORhIedOdeI7ZbY/edit?usp=sharing). **Remember to modify the headers of the taxa (simplify to a single word)**
Once you have the final set of data, copy it into a text file called `bubbleplot_comple.txt`

We are going to plot the abundance results for Desulfobacterota. 

Open R, set the working directory, and run this R.script. 

here is the [R script]

```
library(ggplot2)
library(reshape2)
library(viridis)
library(viridisLite)

pc = read.table("bubbleplot_comple.txt", header = TRUE)
pcm = melt(pc, id = c("ID", "GROUP")) ## GROUP is the column where you put type of sample

pcm$ID <- factor(pcm$ID,levels=unique(pcm$ID))
xx = ggplot(pcm, aes(x = ID, y = variable)) + 
  geom_point(aes(size = value, fill = GROUP), alpha = 0.75, shape = 21) + 
  scale_size_continuous(limits = c(1,10000), range = c(1,10), breaks = c(20,50,70,100,1000, 2000, 5000,10000)) + 
  labs( x= "", y = "", size = "Abundance (Rarified)", fill = "GROUP")  + 
  theme(legend.key=element_blank(), 
        axis.text.x = element_text(colour = "black", size = 7, face = "bold", angle = 90, vjust = 0.3, hjust = 1), 
        axis.text.y = element_text(colour = "black", face = "bold", size = 9), 
        legend.text = element_text(size = 10, face ="bold", colour ="black"), 
        legend.title = element_text(size = 11, face = "bold"), panel.background = element_blank(), 
        panel.border = element_rect(colour = "black", fill = NA, size = 1.2), 
        legend.position = "right", panel.grid.major.y = element_line(colour = "grey95")) +  
        scale_fill_viridis_d() +
        scale_y_discrete(limits = rev(levels(pcm$variable))) 

ggsave(file="bubble_comple.pdf", width = 9, height = 4.5, dpi = 150, units = "in", device = "pdf")


```

![](https://i.imgur.com/psemrXk.png)


![](https://i.imgur.com/Z6Be0ec.png)

See: https://docs.qiime2.org/



https://forum.qiime2.org/t/tutorial-integrating-qiime2-and-r-for-data-visualization-and-analysis-using-qiime2r/4121 


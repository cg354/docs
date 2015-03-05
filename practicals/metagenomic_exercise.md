# Metagenomic analysis 

*2 x 2 hour sessions.*

##Learning Objectives

1.  Reporting bioinformatics projects using Markdown
2.  Data munging with R
3.  FastQC data analysis
4.  Kraken, BWA programs

##Introduction
A recently published paper looked in depth at the environmental metagenome of the NYC subway:  
http://www.sciencedirect.com/science/article/pii/S2405471215000022  

This caused a lot of publicity:
http://www.nytimes.com/2015/02/07/nyregion/bubonic-plague-in-the-subway-system-dont-worry-about-it.html

Then follow-up apologia and discussions:  
http://microbe.net/2015/02/17/the-long-road-from-data-to-wisdom-and-from-dna-to-pathogen/  

Species composition was determined using the megablast-LCA and metaphlan tools.  An alternative approach 
is to use the rapid k-mer based Kraken software. In this exercise we will compare the results from Kraken 
to the published data.

##Write up results using markdown
Make a new git repository for this project.  Clone the repository in any computer you use for analysis 
(EIGC, your own laptop etc.).  Add results files you generate to the git repository and sync by pushing 
and pulling.  **Be careful not to push very large files back to your repo - like fastq files.**  Github 
doesn't like it very much.  

Write up the results of this work in Markdown.  This is best done by directly editing the README on the 
GitHub site and committing changes (editing is activated by pressing the pencil icon on the top right 
corner of the screen). The key is to keep the notebook up to date as you generate the data and perform 
analysis. Then neaten up and write notes at the end.  

Here is a guide to github markdown. 

https://help.github.com/articles/markdown-basics/  

This is an example of how to include an image that is in your github directory.  In this example, just copy the URL of the image on your github repo.

![Sequence quality](https://github.com/IBS574/docs/blob/master/practicals/images/per_sequence_quality.png)

You can also copy and paste Markdown from Rstudio and it will look fine (through it will not process 
code - you need to run through knitr for that). You can also apparently copy in HTML and it will display 
that.  

##Data
I have made sequence data for 4 (out of the more than 1400) samples available on Dropbox.  These data were originally downloaded from NCBI and we have extracted the fastq data.  The data are in zipped folders containing forward and reverse Illumina reads.  Choose one of the four samples to work on.

* https://www.dropbox.com/s/rarrucs0bev1lw0/P00134.zip?dl=0
* https://www.dropbox.com/s/v9rjnqpcy7g0wzz/P00497.zip?dl=0
* https://www.dropbox.com/s/obwp3o18hu3evzg/P00073.zip?dl=0
* https://www.dropbox.com/s/jm0z0smmyi5pxbq/P00070.zip?dl=0

(A side note: the P00134 project actually had two runs, but for simplicity I am not including here the run 
SRR1748707 which produced less data).  

##Part I: Access the data and examine sequence quality
Either go to a terminal on on your local laptop, or log onto EIGC.  If you use EIGC, use the -X option in ssh 
in order to use X-windows visualization later on in the assigment.

     ssh -X <username>@cc3-eigc.emory.edu

Make a directory for your project and go to that directory.  (remeber also to clone your github repo)  

Download the data (note change the file names depending on the sample you choose). 

     curl -o <YOUR_SAMPLE>.zip -L <SAMPLE_LINK_FROM_DROPBOX>

Then unzip the data.  

     unzip <YOUR_SAMPLE>.zip  

This will create a blank directory called __MACOSX which can be cleaned away with  rm -r __MACOSX. 
You can also delete the .zip file is you dont like clutter in your directory.

You can run fastqc on the data.  (Try fastqc -h for more information about controlling output options).

    cd <YOUR_SAMPLE>
    fastqc <SRA_RUN_ACCESSION>_1.fastq.gz <SRA_RUN_ACCESSION>_2.fastq.gz  

This will create directories for each strand with HTML files and images.  Move these results back to your 
git directory and make a short note of the significance in your writeup, making links to appropriate images.   

##Part II: Kraken analysis
Information on Kraken can be found here:

https://github.com/DerrickWood/kraken

Run kraken using the minikraken database.  Go through kraken -h to understand options for this program.  

     kraken --db /home/tread/minikraken_20141208 --fastq-input --gzip-compressed --paired <SRA_RUN_ACCESSION>_1.fastq.gz <SRA_RUN_ACCESSION>_2.fastq.gz --classified-out ./krak_classified_reads > ./kraken_out
     kraken-report --db /home/tread/minikraken_20141208 kraken_out > kraken_report

Take a look through the three output files you have created (krak\_classified\_reads, kraken.out, kraken\_report) 
and try to understand what they are.  The projects may have hits against the biodefense pathogens *Bacillus anthracis* 
and *Yersinia pestis*.  Write a UNIX pipeline or a short ad hoc python script (or combination) that identifies the 
reads that map to these pathogens.  Save them in a FASTA and run a BLAST search against the NCBI database.  Do these 
reads have their best match against the biodefense pathogens, or could they come from close relatives?

_(Hint: it will be helpful to find the NCBI Taxonomy ID associated with each species. This can be found in the 
kraken\_report file. The fourth column is the taxonomic rank ('S' is for species) and the fiftth column is the NCBI 
Tax ID)_

##Part III: Munging the data from supplemental data excel spreadsheet using R
The word *'munge'* appears to have have come into common usage in 
[Scotland and Northern England in the 1940s-1950s](http://english.stackexchange.com/questions/207936/what-is-the-etymology-of-munge), 
as a verb, meaning to munch up into a masticated mess.

To modern data science usage, to *munge* is to find ad hoc solutions to messy formatting problems.

In this case, the supplemental data for the paper contains a valuable excel spreadsheet that summarizes metadata 
about the 1400+ samples and the results of the species assignment using the Metaphlan tool
(http://www.nature.com/nmeth/journal/v9/n8/full/nmeth.2066.html).  This is an unwieldy data set to work with in 
Excel. It is better and more reproducible to bring it in to R and work with it there.  Extracting useful 
information form Excel files is a common time-consuming task in bioinformatics.  

Download the file from 

[Sample Summary](https://www.dropbox.com/s/lca8q8cj2arw8h9/DataTable5-metaphlan-metadata_v19.xlsx?dl=0)

Open in Excel and file as a CSV (comma separated value) test file.  Open a new Rmd file in RStudio and save 
to youe github synced folder.  The data in the Excel file can be impored as a dataframe using the comand below.

     temp.csv <- read.csv("<PATH_TO_CSV_FILE>",stringsAsFactors = FALSE, header = TRUE)
     
You will see that the spreadsheet contained, in effect, two tables.  The first 29 columns describe the metadata 
for each sample (location, GPS, temperature etc).  The rest of the columns describe the percent abundance of 
taxonomic groups dentified by Metaphlan.  The first thought is that the abundances for each row should add up to 
100% but instead they add up to about 800% (how can you show this?).  You realize that this is becasue the 
percentage abundance for each taxonomic rank is shown separately and there are eight taxonomic ranks represented 
(see below).  In order to compare across samples, we only want to focus on one rank.  Lets pick a genus-level 
classification.  A typical record looks like this:

>"k__Bacteria.p__Proteobacteria.c__Gammaproteobacteria.o__Pseudomonadales.f__Pseudomonadaceae.g__Pseudomonas.s__Pseudomonas_sp_TJI_51.t__GCF_000190455"

So, 'k__' means the kingdom rank, 'c__' = class , and so on.  To get the genus level information, you need to 
somehow pick out the columns that contain 'g__' but not 's__'.  

There are numerous ways to do this, R, UNIX, python etc.  Here is a way that illustrates a useful property of grep 
in R and also R set functions.

If you grep for the 'g__' pattern in the columns names, you get a vector of numbers.  These are the columns that 
contain the pattern.

     grep("g__",colnames(temp.csv))
     [1]   35   36   37   38   42   43   44   46   47   48   49   52   53   55   56   57   58   64   65   66   71   72   73
     [24]   74   75   77   78   79   80   82   83   84   85   87   88   89   90   91   92   93   94   95   96   97   98   99
     etc etc
     
You can then use the R 'setdiff' set function to list the columns that contain 'g__' but not 's__'.

     genera <- setdiff(grep("g__",colnames(temp.csv)),grep("s__",colnames(temp.csv)))
     
You can check by looking at the columns of the table corresponding to the numbers using R's subsetting.  You can 
also check that the sums of the proportions add up to 100% (which they do in almost all cases - for this analysis 
we'll not worry about these unusual rows).  Make sure you understand the commands below.

     colnames(temp.csv)[genera]
     apply(temp.csv[genera],1,sum)

Given the above, try and answer the following questions.

1.  Are any genera called present in the Kraken analysis that were not in the metaphlan (or vice versa)? _[QUITE HARD - you will need to google about pattern matching in R]_
2. What were the 20 most common bacterial genus discovered in the study (in terms of the number of samples where the genus was identified as > 0% of the microbiome)?
3.  What was the most common genus found in Brooklyn?
4.  Make a scatterplot of the proportion of the phyla Firmicutes and Bacteroides in each sample.

##Part IV BWA run against a commonly encountered bacterial genomes and qualitative analysis using IGV  
From the previous analysis you will see that *Pseudomonas stutzeri* and *Bacillus cereus* are commonly found 
on the NYC subway surfaces.  Here we will go back and map the metogenome data directly on to refernce genomes 
of these species using the BWA software tool in order to understand the pattern of sequecne reads mapping against 
the individual strains.

Get the fasta files of the chromosmes from NCBI.

     curl "http://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=146280397&rettype=fasta&retmode=text" > Pstutz.fasta
     curl "http://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=30018278&rettype=fasta&retmode=text" > Bcereus.fasta
     
Choose one of these species (or both if you like).  Follow the path below to run the BWA mem alignment program, 
changing names and paths appropriately.  You may want to move the analysis to a new separate directory.

First create an index for the reference fasta file.  This will create the output below.

     bwa index Bcereus.fasta 
     [bwa_index] Pack FASTA... 0.04 sec
     [bwa_index] Construct BWT for the packed sequence...
     [bwa_index] 1.43 seconds elapse.
     [bwa_index] Update BWT... 0.03 sec
     [bwa_index] Pack forward-only FASTA... 0.03 sec
     [bwa_index] Construct SA from BWT and Occ... 0.60 sec
     [main] Version: 0.7.5a-r405
     [main] CMD: bwa index Bcereus.fasta
     [main] Real time: 2.140 sec; CPU: 2.130 sec

This creates some new binary files in your directory.  Just like BLAST indexes , you dont need to worry about them.  
Then run the metagenome fastq files against the index (here my fastq.gz files are in the directory belew so I use 
the '../' prefix - you may have set up your directories differently).

     bwa mem -t 1 Bcereus.fasta ../SRR1748618_1.fastq.gz ../SRR1748618_2.fastq.gz > Bcereus.sam & 
     
The -t option means that only a single processor is used to save resources.  The ampersand (&) means that the process 
runs in the background.  BWA will run for a while spitting out a lot of verbiage.  The results are being saved in a 
SAM format file.  If you are not familiar with SAM and BAM format files, this is a good place to start.  

http://en.wikipedia.org/wiki/SAMtools

The output SAM format file is text. Take a look at it and think of a way to count the number of reads mapped agianst 
the B. cereus genome.  __How does it compare with the results from Kraken and Metaphlan?__

Using samtools the SAM files can be converted to the binary BAM format, which is more compact and customarily used 
for downstream analysis.

     samtools view -bS -o Bcereus.bam Bcereus.sam
     
Then the BAM file is sorted (reads arranged in order of where they align to the referrence) for further downstream 
efficency and indexed.

     samtools sort Bcereus.bam Bcereus_sorted
     samtools index Bcereus_sorted.bam
     
The easist next step is to copy the .bam file, its index (.bai) and the Bcereus fasta file back to a folder on your 
home computer.  Don't sync through git - the file is too large. If you are on a Mac or Linux machine, go to folder in 
your home computer where you want to save the file and use the scp command to retrieve the file.  Below is the command 
I used (with the path to the directories I created).  Windows users see [Suggested SFTP tools](http://ibs574.github.io/docs/setup/#windows-environment). You will need your EIGC pasword.

     scp tread@cc3-eigc.emory.edu:~/Kraken_practical/P00070/bwa_analysis/Bcereus_sorted.bam ./
     scp tread@cc3-eigc.emory.edu:~/Kraken_practical/P00070/bwa_analysis/Bcereus_sorted.bam.bai ./
     scp tread@cc3-eigc.emory.edu:~/Kraken_practical/P00070/bwa_analysis/Bcereus.fasta ./
     
Next is to visualize the aligned reads using the Broad IGV.  The most current version is 2.3.40.  The free software 
can be downloaded from: 

http://www.broadinstitute.org/igv/home

You will need to register your email in order to download.  Open the IGV and then open the 'Genomes' tab.  Click 
the 'Create .genomes file'. Upload the FASTA file for the genome.  Then in the 'File' tab, go to 'Load from file' 
and upload the BAM file.   

Adjust the scale on the top right hand side to view the sequecne reads. Right click on a read and choose the 'View 
as pairs' option. Make sure you under what the arrows and the colors on the pairs mean.

Annotation data, as GFF3 format files can be downleaded at these URLs:

- [*B. cereus* GFF3](https://www.dropbox.com/s/g10i7gtrloe91k9/Bcereus.gff?dl=0)
- [*P. stutzeri* GFF3](https://www.dropbox.com/s/k5qkdkuunc1hjfe/Pstutz.gff?dl=0)

Download to your computer and then load the appropriate GFF3 into the IGV.  You will need to right-click on the 
track and select the Expanded view.

Examine the alignments and make some screen-grabs for your report.  Questions to think about include:
- Is the distribtion of aligned reads even across the genome?
- Are there regions with unusually high or low coverage?  
- Are there any 'broken' mate-pairs and why might this be happening.  
- In general, are there a large number of SNPs between the individual reads and the genome?  
- Is there any evidence that the metagenome reads derive from more than one genetically distinct organism?

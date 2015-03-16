#*De novo* assembly

In this practical you will create synthetic fastq files based on a real genome sequence that mimics the output of an Illumina sequencing instrument.  In this way you can examine how errors affect de novo assembly efficiency.  

##Learning Objectives

1.	Creating synthetic sequencing data to preview experiments.
2.	*de novo* assembly with Velvet.
3.	Looking at assembly statistics.
4. 	Understanding how sequence error affects assembly.
5.	Visualizing assembly graphs.

##Walkthrough

To start, log on to cc3-eigc, create a new directory for your analysis and cd to that directory.  Download the *S. aureus* strain N315 chromosome.  

	curl -L https://www.dropbox.com/s/xvt0xzmjgbepfle/N315.fasta?dl=0 > N315.fasta

To make the synthetic Illumina reads, we will use the ART software. Details of the ART program can be found [here](http://www.niehs.nih.gov/research/resources/software/biostatistics/art/)

The options for running the software can be viewed by typing,  

	art_illumina

Now, generate 35x coverage with 100 bp paired ends using ART.  Make sure you understand all the parameter options.  Should take about 40s.

	art_illumina -i N315.fasta -p -l 100 -f 35 -m 500 -s 10 -o Sa_35x-pe100 -ef -na
	
Look at the output of the fastq file.

	less Sa_35x-pe1001.fq
	
How many reads are there in each file?  

You can visualize with fastqc if you like.  Run the following and then scp the zipped file to your computer to visualize.

	fastqc Sa_35x-pe1001.fq -noextract
	
Next, we want to de novo assemble the reads we have created.  Velvet was the first effective de novo assembler for short reads.  Its a little old-fashioned now but its good for rapid analysis.
Velvet has two parts: velveth makes the kmer hashes and velvetg creates the de Bruijn graphs.

	velveth -h
	
As you can see, velveth accepts data in multiple formats, including fastq, BAM etc.  For convenience, we will use the SAM format file created by ART.

	velveth Sa_35x-pe100_hash 31 -shortPaired -sam Sa_35x-pe100.sam

You will notice a new directory has been created for the hash sequences.   Next, look at velvetg

	velvetg --help
	
Here is a simple way to run velvetg on the data.

	velvetg Sa_35x-pe100_hash -cov_cutoff auto -ins_length 500 -clean yes -exp_cov 35

This writes output to the hash directory you created.  You will see a 'contigs.fa' and 'stats.txt'.  The first is the sequences of the assembly and the second is a set of stats about each contig?

	ls Sa_35x-pe100_hash
	contigs.fa  Graph  Log  PreGraph  Sequences  stats.txt
	
Notice that velvetg outputs the N50 estimation and other stats at the end of its run.
The descriptor line for each contig (called a 'node' in contig.fa) gives, length and coverage.  Can you use UNIX command line tools to quickly do the following:

1.	Find the coverage of the longest contig?
2.	Calculate the total length of all the contigs with coverage > 30?
3.	Find average contig length with coverage > 30?

Repeat the assembly and evaluation of the the SAM file with no errors.  How much better is this data? Why can you not reassemble the S. aureus genome efficiently from these synthetic data?

##For fun: assembly visualization.

Download the [Bandage software](http://rrwick.github.io/Bandage/) on your local laptop. On your Mac you may have to change security setting in you Systems preferences.

Use scp to download the 'Graph' file produced by velvetg to your local computer. Load this graph into Bandage.  

Use the "Draw graph' option to view the graph and select 'Color by coveage'.  What is the graph showing you?

![Screenshot of N315 contig graph in bandage](https://github.com/IBS574/docs/blob/master/docs/img/bandage_view.png)




	




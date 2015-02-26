# Learning more about BLAST

BLAST is not just about the NCBI web site. The software can be easily installed on your local machine - see the [BLAST installation guide](http://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE_TYPE=BlastDocs&DOC_TYPE=Download).  Most Bioinformatics servers have BLAST installed on the default path.  If you are adventurous, you can easily start your own an AWS instance as a  BLAST server with a custom database.  

Using the command line BLAST has some advantages to the web site.  Its easy to make your own databases and its also easy to make some simple but powerful custom analysis pipelines.

##Learning Objectives

1. Downloading data using wget/curl
2. Making custom BLAST databases
3. Saving results in tab format and post-processing with UNIX
4. Effect of modifying -word_size and -evalue parameters
 

## Walkthrough
Log onto to the cc3-eigc.emory.edu server

    ssh <your_username>@cc3-eigc.emory.edu
    mkdir blast_exercise
    cd blast_exercise

Download some sequences that will be used to make a database. In this case a set of complete whole genome sequences of the bacterium *Bacillus cereus* sensu lato.  These were downloaded  earlier from NCBI using the Entrez query,  

>xid1396[Organism] AND srcdb_refseq[prop] AND "complete genome" NOT "whole genome shotgun"

Downoad using curl (or wget if curl is not available).  Make sure you understand the -O and -L options.  

    curl -O -L  https://www.dropbox.com/s/6g67pyr04rcrcr7/Bcereus_complete_genomes.fasta

This will take a few minutes to download 123 Mbytes (you can follow the progress while the file is downloading). You can check to see what you have using the grep command.

    grep '>'   Bcereus_complete_genomes.fasta

How can you easily count the number of the sequences in the file?

Before performing a blast search you need to create a database.  Use the following command,  

    makeblastdb -in Bcereus_complete_genomes.fasta -input_type 'fasta' -dbtype 'nucl'

This will create some new binary files with .nsq, . nhr and .nin suffices that you dont have to worry about. To get detailed information about running makeblastdb, use the -help.

    makeblastdb -help

In order to get a query sequence, we will download the anthrax lethal toxin gene using an NCBI Entrez eutils query.  

    curl "http://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=390132838&rettype=fasta&retmode=text" > lef.fasta

Take a look at the nucelotide sequence using the UNIX *more* command.

Now perform a simple BLAST database search, using the minimal parameters.

    blastn -db Bcereus_complete_genomes.fasta -query lef.fasta

This outputs the results to STDOUT.  How would you change the settings to capture the alignment as a file?  What is interesting biologicaly about this result?

There a LOT of options for customization of the BLAST search. Check them out using the -help option.

    blastn -help

First thing we are going to do is change the output format. We are going to use a tabular output that it very convenient for parsing (and once you are used to it, also easier to read quickly than the normal alignment view).

    blastn -db Bcereus_complete_genomes.fasta -query lef.fasta -outfmt 6
    gi|390132838|gb|JQ798176.1|	gi|749295517|ref|NZ_CP009941.1|	98.44	2430	38	0	1	2430	1316886	1314457	0.0	4277

The 12 columns are as follows, 

    query id, subject id, % identity, alignment length, mismatches, gap opens, q. start, q. end, s. start, s. end, evalue, bit score
    
Download another query sequence, this time a multi-fasta file of all genes on the *B. anthracis* pXO2 plasmid.   

    curl -L https://www.dropbox.com/s/ea8s8d4dn7zoqux/pXO2_genes.fasta.txt > pXO2.fasta
    
Take a look at the file and work out how many genes there are.  

We can do a multiple blast search against the database.  

    blastn -db Bcereus_complete_genomes.fasta -query pXO2.fasta -outfmt 6
    lcl|NC_007323.3_cds_WP_000281522.1_37	gi|217957581|ref|NC_011658.1|	90.94	331	30	0	1	331	524371	524701	2e-124	 446
    lcl|NC_007323.3_cds_WP_000281522.1_37	gi|217957581|ref|NC_011658.1|	90.94	331	30	0	1	331	689702	689372	2e-124	 446
    lcl|NC_007323.3_cds_WP_000281522.1_37	gi|217957581|ref|NC_011658.1|	90.94	331	30	0	1	331	1565195	1565525	2e-124	 446
    lcl|NC_007323.3_cds_WP_000281522.1_37	gi|217957581|ref|NC_011658.1|	90.94	331	30	0	1	331	1654634	1654304	2e-124	 446
    lcl|NC_007323.3_cds_WP_000281522.1_37	gi|217957581|ref|NC_011658.1|	90.94	331	30	0	1	331	2974794	2975124	2e-124	 446
    lcl|NC_007323.3_cds_WP_000281522.1_37	gi|217957581|ref|NC_011658.1|	90.94	331	30	0	1	331	3782150	3781820	2e-124	 446
    etc, etc ...

You can see why the tab-delimited output start to seem much more managable for these types of results. There are some very quick and simple ways to start to make sense of this.  First question is, how many distinct pXO2 genes have matches at the E-value cutoff against this database?  Its easy with a combination of awk, sort, wc and UNIX pipes.

    blastn -db Bcereus_complete_genomes.fasta -query pXO2.fasta -outfmt 6 | awk '{print $1}' | sort -u | wc -l
      13

Make sure you understand how this magic was done!  Note - becasue this BLAST search is very fast I am rerunning the pipeline from the start each time for convenience. If you have a slow search, you may want to save results and run the output through the pipeline.

Ok - can you modify the pipleine to count,

1.  the number of sequences in the datasbe that had at least one hit?
2.  The number of matches with a sequence identity > 98%?

##Exercises 

Examine the effect that changing the key parameters on the number and quality of the results. Specifically,

1.  -word_size  values 5, 15, 25, and 40
2.  -evalue values  1, 1e-10, 1e-20
  
What combination of parameters would you use to identify only orthogs with a match that spans > 90% of the query gene?

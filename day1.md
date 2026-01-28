#What I learned today

testing to sync

**DAY1**
1. **differnce between `GUI` and `CLI`** 
`GUI` Graphical User Interface -> easy to use, especially for berginners, but less efficient and slow
`CLI` Command Line Interface -> fast and efficient, you can chain commands and automate, but not that beginner friendly

2. **Linux commands**
`pwd` for current path working directory
`cd` for chaning the working directory
`mv` for moving a directory
`mkdir` for creating a directory; with `-p` there is no problem if there are directories with the same name
`ls` for listing files in a directory
`cp` for copying files
`*` to select everything with something
`rm` for removing files
`rm -r []/` for removing directories
`cat` for editing files, `cat >` for overwrite something, `cat >>` for adding something

*tab* to autofil

3. **bash script**
*Terminal:`ssh -X sunam232@caucluster.rz.uni-kiel.de` to connect to cluster, tipping in the password
* difference in `$HOME` and `$WORK` -> only work in `$WORK`
* `micromamba activate [...]`: to acitivate specific commands, if not acitivated it may not work
* `sbatch` to upload bash script, `squeue` to see how long it takes, if it isn't already finished 
* Script: *strg+s* to safe it
* Loops in bash script: used to do something and apply it on several selected f.e. files
* `#` to create comments

**DAY2** 
1. Quality control of raw reads (before proceeding further analyses)
* `fastq` to get an overview, quality control of the reads
-> the reads are not perfect, you have to control
-> Sequence Length of each read: 20-301
#`mkdir -p day2`
#`cd $WORK/day2`
#`mkdir -p fastqc_output`
-> command: `fastqc ../metagenomics/0_raw_reads/*.fastq.gz -o fastqc_output`
-> `-o` putting the contents of the specific file to the folder, which was created before
* `fastp` specify different input files: forwar (R1) and reverse (R2)
-> command: fastp -i sample1_R1.fastq.gz -I sample1_R2.fastq.gz -o outdir/sample1_R1_clean.fastq.gz -O outdir/sample1_R2_clean.fastq.gz -t 6 -q 20 -h sample1.html -R "Sample 1 Fastp Report"
-> with *20* in command we decide that we wanted to trimm everything below 20, but there was nothing
-> we remove first 6 bases to remove the ?Begriff
-> then some other naming things for us
* 3x
#mkdir -p fastp_output
#fastp -i ../metagenomics/0_raw_reads/BGR_130305_mapped_R1.fastq.gz -I ../metagenomics/0_raw_reads/BGR_130305_mapped_R2.fastq.gz -o fastp_output/BGR_130305_fastp_cleaned_R1.fastq.gz -O fastp_output/BGR_130305_fastp_cleaned_R2.fastq.gz -t 6 -q 20 -h _130305.html -R _130305
#fastp -i ../metagenomics/0_raw_reads/BGR_130527_mapped_R1.fastq.gz -I ../metagenomics/0_raw_reads/BGR_130527_mapped_R2.fastq.gz -o fastp_output/BGR_130527_fastp_cleaned_R1.fastq.gz -O fastp_output/BGR_130527_fastp_cleaned_R2.fastq.gz -t 6 -q 20 -h _130527.html -R _130527
#fastp -i ../metagenomics/0_raw_reads/BGR_130708_mapped_R1.fastq.gz -I ../metagenomics/0_raw_reads/BGR_130708_mapped_R2.fastq.gz -o fastp_output/BGR_130708_fastp_cleaned_R1.fastq.gz -O fastp_output/BGR_130708_fastp_cleaned_R2.fastq.gz -t 6 -q 20 -h _130708.html -R _130708

2. Assembly

#genome assemmbly with megahit

#megahit -1 fastp_output/BGR_130305_fastp_cleaned_R1.fastq.gz -1 fastp_output/BGR_130527_fastp_cleaned_R1.fastq.gz -1 fastp_output/BGR_130708_fastp_cleaned_R1.fastq.gz -2 fastp_output/BGR_130305_fastp_cleaned_R2.fastq.gz -2 fastp_output/BGR_130527_fastp_cleaned_R2.fastq.gz -2 fastp_output/BGR_130708_fastp_cleaned_R2.fastq.gz -o megahit_assemblies --min-contig-len 1000 --presets meta-large -m 0.85 -t 12 

-> megahit puts the smaller sequences together, which fit together -> to get a longer sequence
-> but with lumina reads its not that possible
-> fasta like grpah was created from a normal fasta file -> useful for bandage
-> In Bandage we see the visualization of the sequences we put together
-> larger ones and smaller ones -> more reads were put together in the larger ones -> more likely correct and better quality
-> now they can be used for more metagenomic work

**DAY3**

1. Check assembly quality
`metaquast day2/megahit_assemblies/final.contigs.fa -o day3 -t 6 -m 1000`

* What is your N50 value? Why is this value relevant?
`3014` -> `a higher value shows a good genome assembling. So it stands for the quality.`
* How many contigs are assembled?
`55835`
* What is the total length of the contigs?
`142642168 bp`

2. Map sequencing reads to contigs

3. Bin contigs into genomes (MAGs) based on read mapping

* How many A R C H A E A bins did you get from MetaBAT2?
* How many A R C H A E A bins did you get from Maxbin2?

4. Estimate the quality of binned MAGs

* Which binning strategy gives you the best quality for the A R C H A E A bins?
* How many A R C H A E A bins do you get that are of High quality?
* How many B A C T E R I A bins do you get that are of High quality?





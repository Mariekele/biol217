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
-> quast is a very informative QUality ASsessment Tool to evaluate genome assemblies
code: `metaquast day2/megahit_assemblies/final.contigs.fa -o day3 -t 6 -m 1000`

* What is your N50 value? Why is this value relevant?
`3014` -> `a higher value shows a good genome assembling. So it stands for the quality. The higher the value the fewer and longer the reads are. The genome is more complete (?)`
-> the bigger the number more material is covered by longer contigs
-> 3000 is very fragmentet
* How many contigs are assembled?
`55835`
* What is the total length of the contigs?
`142642168 bp`

2. Map sequencing reads to contigs
-> Read mapping results allow us to check for contamination, classify taxa, identify incorrect binning, and more.
-> first clean up the assembly and convert it into a more efficient format before map the reads onto it.
 1. Re-formatting the contigs: To simplify sequence IDs and filter out short contigs, we will use `anvi-script-reformat-fasta`
code: `anvi-script-reformat-fasta day2/megahit_assemblies/final.contigs.fa -o day3/final.contigs.simplified.fa --min-len 1000 --simplify-names --report-file day3/table.txt`

 2. Indexing the contigs
 -> after organzied and indexed contigs, the mapping runs faster
 -> Bowtie 2 is an ultrafast and memory-efficient tool for aligning sequencing reads to long reference sequences.
 command: `bowtie2-build day3/final.contigs.simplified.fa day3/contigs.anvio.fa.index`

 3. Mapping reads onto contigs
 -> code: `bowtie2 -1 day2/fastp_output/BGR_130305_fastp_cleaned_R1.fastq.gz -2 day2/fastp_output/BGR_130305_fastp_cleaned_R2.fastq.gz -S day3/BGR_130305.sam -x day3/contigs.anvio.fa.index  --very-fast`

 `bowtie2 -1 day2/fastp_output/BGR_130527_fastp_cleaned_R1.fastq.gz -2 day2/fastp_output/BGR_130527_fastp_cleaned_R2.fastq.gz -S day3/BGR_130527.sam -x day3/contigs.anvio.fa.index  --very-fast` 

 `bowtie2 -1 day2/fastp_output/BGR_130708_fastp_cleaned_R1.fastq.gz -2 day2/fastp_output/BGR_130708_fastp_cleaned_R2.fastq.gz -S day3/BGR_130708.sam -x day3/contigs.anvio.fa.index  --very-fast`

 -> convert them into machine language -> make data processinf much faster:`Sequence Alignment Map (.sam) to Binary Alignment Map (.bam)`
 `samtools view -Sb day3/BGR_130305.sam > day3/BGR_130305.bam`
 `samtools view -Sb day3/BGR_130527.sam > day3/BGR_130527.bam`
 `samtools view -Sb day3/BGR_130708.sam > day3/BGR_130708.bam`

 4. Sorting mapped reads
 -> speeds up data processing and allows to do downstream analyses like visualization and variant calling
 -> with `anvi-init-bam` sort and indexes the `.bam` files in just one command
 `anvi-init-bam day3/BGR_130305.bam -o day3/BGR_130305_sort.bam`
 `anvi-init-bam day3/BGR_130527.bam -o day3/BGR_130527_sort.bam` 
 `anvi-init-bam day3/BGR_130708.bam -o day3/BGR_130708_sort.bam`


3. Bin contigs into genomes (MAGs) based on read mapping
-> to figure out which microbes are present in the samples
 1. Generating contigs database
 `anvi-gen-contigs-database` to store many types of information; compute k-mer frequencies for each contig; Soft-split contigs longer than 20,000 nucleotides into smaller ones; dentify open reading frames (ORFs) using Prodigal (a program for predicting genes in bacteria and Archaea)
 `anvi-gen-contigs-database -f day3/final.contigs.simplified.fa -o day3/contigs_database.db -n projectday3`

 2. Annotating ORFs
 -> with `anvi-run-hmms` searching for any potential biological functions that the predicted ORFs may have
 `anvi-run-hmms -c day3/contigs_database.db --num-threads 4`

 3. Visualizing the contigs database
 command: `anvi-display-contigs-stats day3/contigs_database.db` -> **only in terminal not with bashscript!!!**
 -> **Archea:**
 -> **Bacteria:**
 -> **Protista:**
 
 4. Creating an `anvi'o` profile
 -> like an upgraded `anvi'o` database hat can also store read mapping results and detailed per-nucleotide information
 -> code:
 `anvi-profile -i day3/BGR_130305_sort.bam -c day3/contigs_database.db --output-dir day3/sample1` 
 `anvi-profile -i day3/BGR_130527_sort.bam -c day3/contigs_database.db --output-dir day3/sample2`
 `anvi-profile -i day3/BGR_130708_sort.bam -c day3/contigs_database.db --output-dir day3/sample3`

 5. Merging `anvi'o` profiles from all samples
 -> merge the 3 different samples to one profile to analyze and compare them
 -> `--enforce-hierarchical-clustering`: Construct a phylogenetic tree that shows the relationships between the contigs.
 `anvi-merge day3/sample1/PROFILE.db day3/sample2/PROFILE.db day3/sample3/PROFILE.db -o day3/merged_profiles -c day3/contigs_database.db --enforce-hierarchical-clustering`

 6. Binning contigs into genomes 
 -> using the excat same commands for both, just exchanging `METABAT2` and `MaxBin2` 
 * **Using MetaBAT2**
 `-p` merged profile follows
 `-c` contigs database follows
 
 `anvi-cluster-contigs -p day3/merged_profiles/PROFILE.db -c day3/contigs_database.db -C METABAT2 --driver metabat2 --log-file day3/metabat2_logfile --just-do-it` 

 `anvi-summarize -p day3/merged_profiles/PROFILE.db -c day3/contigs_database.db -o day3/metabat2_sum -C METABAT2`


 * **Using MaxBin2**

 `anvi-cluster-contigs -p day3/merged_profiles/PROFILE.db -c day3/contigs_database.db -C MAXBIN2 --driver maxbin2 --log-file day3/maxbin2_logfile --just-do-it` 
 
 `anvi-summarize -p day3/merged_profiles/PROFILE.db -c day3/contigs_database.db -o day3/maxbin2_sum -C MAXBIN2`


-> data can be seen in the `.html` file, but each of the **MAGs** have one `fasta`file.
* How many A R C H A E A bins did you get from MetaBAT2?
-> `3`
* How many A R C H A E A bins did you get from Maxbin2?
-> `1`



4. Evaluating MAGs Quality
for the following steps only bins generated from **MetaBAT2** is used
-> evaluate how complete and pure each of the bin (MAG) is with `anvi-estimate-genome-completeness`
-> command: `anvi-estimate-genome-completeness -c day3/contigs_database.db -p day3/to/merged_profiles/PROFILE.db -C METABAT2`
-> and then to check what bin collections were generated:
-> command: `anvi-estimate-genome-completeness --list-collections -p day3/merged_profiles/PROFILE.db -c day3/contigs_database.db`

4. Estimate the quality of binned MAGs

command: `anvi-interactive -p day3/merged_profiles/PROFILE.db -c day3/contigs_database.db -C METABAT2`
-> `anvi-interactive` to manually inspect and work on the bins



* Which binning strategy gives you the best quality for the A R C H A E A bins?
-> MetBat is better because there are 3 archaea and one of them has a good quality, but in MaxBin theres only one archaea which hasn´t a good quality because of the high Redundance -> **the best Archaea is in METBAT__27**
* How many A R C H A E A bins do you get that are of High quality?
-> I get one archaea of a good quality
* How many B A C T E R I A bins do you get that are of High quality?
-> 12 of good quality

conclusion: on day3 we binned assembled contigs into genome bins and checked the bins to see if they represent bacteria or Archaea.

**DAY4**
Aim: Examine the bins from day3 and improve their quality

1. Evaluating MAGs Quality -> did it on day3
2. Refining Archaea bins
 -> work with a copy of the data, but first find the bins  which contains the Archaea
 -> METABAT__27
 -> METABAT__36
 -> METABAT__7
 then copy them into day4:
 `cp -r day3/metabat2_sum/bin_by_bin/METABAT__27 day4`
 `cp -r day3/metabat2_sum/bin_by_bin/METABAT__36 day4`
 `cp -r day3/metabat2_sum/bin_by_bin/METABAT__7 day4`

 now refine them:
 day4/
└── refine/
    ├── METABAT__27/
    │   └── METABAT__27-contigs.fa
    ├── METABAT__36/
    │   └── METABAT__36-contigs.fa
    └── METABAT__7/
        └── METABAT__7-contigs.fa

 1. Detecting chimeras in MAGs
 -> with **GUNC** checking for chimeras and potential contamination **(Chimeric genomes are genomes wrongly assembled out of two or more genomes coming from seperate organisms)**
 -> chaning environment to `00_gunc` and making directoires to store the output and then run to check:
 commands: 
 `gunc run -i day4/METABAT__27/METABAT__27-contigs.fa -r $WORK/databases/gunc/gunc_db_progenomes2.1.dmnd --out_dir day4/METABAT__27/gunc_out_27 --detailed_output --threads 12`
 
 `gunc run -i day4/METABAT__36/METABAT__36-contigs.fa -r $WORK/databases/gunc/gunc_db_progenomes2.1.dmnd --out_dir day4/METABAT__36/gunc_out_36 --detailed_output --threads 12`
 
 `gunc run -i day4/METABAT__7/METABAT__7-contigs.fa -r $WORK/databases/gunc/gunc_db_progenomes2.1.dmnd --out_dir day4/METABAT__7/gunc_out_7 --detailed_output --threads 12`

 2. Creating interactive plots of the chimeras
 -> so in the step before we runned the chimera detection and now we are going to visualize the results in a plot

 `gunc plot -d day4/METABAT__7/gunc_out_7/diamond_output/METABAT__7-contigs.diamond.progenomes_2.1.out -g day4/METABAT__7/gunc_out_7/gene_calls/gene_counts.json --out_dir day4/METABAT__7/gunc_out_7`

 `gunc plot -d day4/METABAT__36/gunc_out_36/diamond_output/METABAT__36-contigs.diamond.progenomes_2.1.out -g day4/METABAT__36/gunc_out_36/gene_calls/gene_counts.json --out_dir day4/METABAT__36/gunc_out_36`

 `gunc plot -d day4/METABAT__27/gunc_out_27/diamond_output/METABAT__27-contigs.diamond.progenomes_2.1.out -g day4/METABAT__27/gunc_out_27/gene_calls/gene_counts.json --out_dir day4/METABAT__27/gunc_out_27`
 
 **Questions:**
 *  Do you get A R C H A E A bins that are chimeric?
 H i n t : Look at the CSS score and the column "PASS GUNC" in the gunc output tables for each bin.
 
 `clade_separation_score(CSS):` a result of applying a formula explained in GUNC paper to taxonomy and contig labels of genes retained in major clades. Ranges from 0 to 1 and is set to 0 when genes_retained index is <0.4 because that is too few genes left.
 -> if its high, so above 0.45 its a hint for beeing chimeric
 
 `pass.GUNC:` If a genome passes GUNC analysis it means it is likely to not be chimeric (or that chimerism cannot be detected especially when its reference representation (RRS) is low). A genome passes if clade_separation_score <= 0.45, a cutoff benchmarked using simulated genomes.


 -> `Metabat7:`
 SSC: is 0 everywhere except in species taxonomic level there is it 0.1 -> means its not chimeric 
 -> gunc analyses passes everywhere so it is likely to not be chimeric in any way

 -> `Metabat36:`
 SSC: is 0 everywhere except in species ther it is 0.78 which means it is likely to be chimeric
 -> gunc analyses only fails in taxonomic level of species and passes everywhere else -> means only the species are different

 -> `Metbat7:`
 SSC: is 1 in every taxonomic level except in species theres 0.4
 -> and only in species it passes the gunc analyses which means it is likely to not be chimeric or it can´t be detected
 -> both outputs are connected


 * In your own words, briefly explain what a chimeric bin is
 -> it means that some parts are falsy connected together, which do not really belong together. Reads of different organisms were put together and are giving wrong connections.

3. Manual bin refinement
-> As large metagenome assemblies can result in hundreds of bins, pre-select some of the better ones for manual refinement, e.g., > 70% completeness

-> first create a copy to avoid to destroy important data
`cp day3/merged_profiles/PROFILE.db day4/PROFILE_refined.db`

now `anvi-refine` to work on my bins manually

command: `anvi-refine -c day3/contigs_database.db -p day4/PROFILE_refined.db --bin-id METABAT__7 -C METABAT2` 

`anvi-refine -c day3/contigs_database.db -p day4/PROFILE_refined.db --bin-id METABAT__27 -C METABAT2`
-> **using only that one**

`anvi-refine -c day3/contigs_database.db -p day4/PROFILE_refined.db --bin-id METABAT__36 -C METABAT2`

-> Screenshots after sorting for content mean

**Questions:**
* How much could you improve the quality of your A R C H A  E A ?
  Compare the completeness and redundance of the bin before and after refining.
  -> **in the plot:**
->  Domain	Domain Confidence	HMM Source	Completion	Redundancy
✓	archaea	0.98	Archaea_76	98.68%	2.63%
-> **in the .html:**
Bin METABAT__27
Source metabat2 
Taxonomy N/A 
Total Size 1.82Mb
Num Contigs 262 
N50 8,484
GC Content 59.39%
Compl. 98.68%
Red. 	2.63% 
SCG Domain archaea

-> **nothing changed between before and after refining**


4. Visualizing Coverage
-> to see how abundant the Archaea MAGs really are

`anvi-interactive -p day3/merged_profiles/PROFILE.db -c day3/contigs_database.db -C METABAT2`

-> only looking for 27:
Item Name	METABAT__27
Layer Name	source
length	1820850
gc_content	0.5938968850137224
BGR_130305_sort	8.012879472552424
BGR_130527_sort	5.2168197907991685
BGR_130708_sort	3.48495948804213
percent_completion	98.6842105263158
percent_redundancy	2.6315789473684212
matching_domain	archaea (1.0)
bin_name	METABAT__27
source	metabat2

-> for 7:
Item Name	METABAT__7
Layer Name	source
length	446145
gc_content	0.4370266251731875
BGR_130305_sort	3.855132063770183
BGR_130527_sort	0.04694284515131497
BGR_130708_sort	2.3668113692812196
percent_completion	38.1578947368421
percent_redundancy	0
matching_domain	blank (0.3)
bin_name	METABAT__7
source	metabat2

-> for 36:
Item Name	METABAT__36
Layer Name	source
length	1317425
gc_content	0.5990973899156462
BGR_130305_sort	3.7029119659339633
BGR_130527_sort	1.2675120674322242
BGR_130708_sort	2.3894890684267542
percent_completion	48.68421052631579
percent_redundancy	9.210526315789474
matching_domain	blank (0.5)
bin_name	METABAT__36
source	metabat2



**Questions:**
* How abundant (relatively) are the A r c h a e a bins in the 3 samples?
-> Only METABAT__27 is archaeal.
Relative abundance across samples:

BGR_130305 > BGR_130527 > BGR_130708
(≈ 48%, 31%, 21%, respectively)

Bins 7 and 36 aren’t confidently archaeal.


**DAY5**

1. Taxonomic assignment
-> adding taxonomic annotations to the MAGs
`anvi-run-scg-taxonomy`:  associates the single-copy core genes in the contigs-db with taxnomy information. 

command: `anvi-run-scg-taxonomy -c day3/contigs_database.db -T 20 -P 2`

-> This program makes quick taxonomy estimates for genomes, metagenomes, or bins stored in your contigs-db using single-copy core genes. But using the program in metagenome-mode, because the contigs contain multiple genomes

commands:
-> to estimate abundance of Ribisomal RNAs within the dataset: for each sample and `> temp.txt` to save

`anvi-estimate-scg-taxonomy -c day3/contigs_database.db -p day3/sample1/PROFILE.db --metagenome-mode --compute-scg-coverages --update-profile-db-with-taxonomy > temp.txt`
`anvi-estimate-scg-taxonomy -c day3/contigs_database.db -p day3/sample2/PROFILE.db --metagenome-mode --compute-scg-coverages --update-profile-db-with-taxonomy > temp.txt`
`anvi-estimate-scg-taxonomy -c day3/contigs_database.db -p day3/sample3/PROFILE.db --metagenome-mode --compute-scg-coverages --update-profile-db-with-taxonomy > temp.txt`

-> ONE final summary to get comprehensive info about your METABAT2 bins:

`anvi-summarize -p day3/merged_profiles/PROFILE.db -c day3/contigs_database.db -o day5/SUMMARY_METABAT2_FINAL -C METABAT2`

-> now the taxonomy is added. You can look at `index.htlm` and see which MAGs belong to which species


**Questions**
*  Did you get a species assignment to the A R C H A E A bins previously identified?
-> yep
-> Metabat__27: Methanocellus sp012797575 -> so we found out the genus, but not the specific species
-> Metabat__26: Methanocellus thermohydrogenotrophicum -> so we found out the exact species
-> Metabat__7: Methanosarcina flavescens -> so we found out the exact species

* Does the HIGH-QUALITY assignment of the bin need revision?
-> yep, should be revisied. **But why?**

* hint: MIMAG quality tiers https://www.nature.com/articles/nbt.3893

2. Genome dereplication (BONUS)
-> `anvi-dereplicate-genome` will use `fastANI` to perform the dereplication. It calculates the `Average Nucleotide Identity`, its faster than the `piANI`. And its going to remove the bacteria of the archaea.







* How many species do you have in the dataset?
->

* Try to dereplicate again at 90% identity then at 80%identity. In your own words, explain the differences between the different %identities.
-> 








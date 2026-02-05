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


-> [picture](/image.png)

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
-> [METABAT7](/METABAT7.png)
-> [METABAT27](/METABAT27NEW.png)
-> [METABAT36](/METABAT36.png)

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
-> Metabat__26: Methanocellus thermohydrogenotrophicum -> so we found out the exact species -> **The High-quality bin**
-> Metabat__7: Methanosarcina flavescens -> so we found out the exact species

* Does the HIGH-QUALITY assignment of the bin need revision?
-> no is fine as it is
-> completeness nearly 100%
-> genome nearly complete
-> taxonomy coherent
-> the markers agree
-> Bin was okay

* hint: MIMAG quality tiers https://www.nature.com/articles/nbt.3893

2. Genome dereplication (BONUS)
-> `anvi-dereplicate-genome` will use `fastANI` to perform the dereplication. It calculates the `Average Nucleotide Identity`, its faster than the `piANI`. And its going to remove the bacteria of the archaea.

-> i created tabfile.csv:
27	METABAT__27	METABAT2	day3/merged_profiles/PROFILE.db	day3/contigs_database.db	day4/METABAT__27/METABAT__27-contigs.fa
7	METABAT__7	METABAT2	day3/merged_profiles/PROFILE.db	day3/contigs_database.db	day4/METABAT__7/METABAT__7-contigs.fa
36	METABAT__36	METABAT2	day3/merged_profiles/PROFILE.db	day3/contigs_database.db	day4/METABAT__36/METABAT__36-contigs.fa


command: `anvi-dereplicate-genomes -i tabfile.csv --program fastANI --similarity-threshold 0.95 -o ANI --log-file log_ANI -T 10`
but that was only for the archaea
```
anvi-dereplicate-genomes -i tabfilenewnew.txt --program fastANI --similarity-threshold 0.95 -o ANI2 --log-file log_ANI -T 10 --force-overwrite
anvi-dereplicate-genomes -i tabfilenewnew.txt --program fastANI --similarity-threshold 0.90 -o ANI90 --log-file log_ANI -T 10 --force-overwrite
anvi-dereplicate-genomes -i tabfilenewnew.txt --program fastANI --similarity-threshold 0.80 -o ANI80 --log-file log_ANI -T 10 --force-overwrite
```
-> **remember fot the exam what I did?**


-> for every bin. 95% identity, 90% identity, 80% identity


* How many species do you have in the dataset?
-> 46 species

* Try to dereplicate again at 90% identity then at 80%identity. In your own words, explain the differences between the different %identities.
-> nothing changes with 95% and 90% identity. But with 80% I got only 44 species. When I look into the Cluster report it is shown that 2 species are the same. Its bin27 and bin36 which were the archaea. They were actually from the same genus. So why do we get different bins, maybe its because some contigs were falsley compared together or maybe because the binning isnt looking for the nucelotid identity. They just looking for the k-mers and somehting else. Those are less sensitive than nucleotid identity. There are a lot of technological reasons for miss binnig. 
And bin14 and bin42 which where unnamed bacteria.

-> [picture](/image.png) -> zum Bilder abspeichern 


**DAY6**
- working in the micromamba short reads or long reads

1. **Data today**

2. **Quality Control**
 1. <Short reads>
 - run **fastqc**
 ```
 mkdir -p $WORK/genomics/1_short_reads_qc/1_fastqc_raw
 for i in genomics/0_raw_reads/short_reads/*.gz
 do 
    fastqc $i -o genomics/1_short_reads_qc/1_fastqc_raw -t 8
  done
  ```

 - run **fastp**
 `mkdir -p $WORK/genomics/1_short_reads_qc/1_fastp_out`
 
 `fastp -i genomics/0_raw_reads/short_reads/241155E_R1.fastq.gz -I genomics/0_raw_reads/short_reads/241155E_R2.fastq.gz -o genomics/1_short_reads_qc/cleaned_241155E_R1.fastq.gz -O genomics/1_short_reads_qc/cleaned_241155E_R2.fastq.gz -t 6 -q 20 -h genomics/1_short_reads_qc/1_fastp_out/fastp_report.html -R genomics/1_short_reads_qc/1_fastp_out/fastp_report`

 - Check the quality of the cleaned reads with fastqc again
 
 `mkdir -p $WORK/genomics/1_short_reads_qc/fastqc_cleaned`
 ```
 for i in genomics/1_short_reads_qc/*.gz
 do 
   fastqc $i -o genomics/1_short_reads_qc/fastqc_cleaned -t 16
  done
  ```
  **Questions**
 * *How Good is the read quality?*
   -> before: average 35
   -> after: average 36
 * *How many reads before trimming and how many do you have now?*
   -> before: 1639549
   -> after: 1613392
 * *Did the quality of the reads improve after trimming?*
   -> the quality improved slightly from 35 to 36


 2. <Long reads>
 - *NanoPlot:*
 `mkdir -p genomics/2_long_reads`
 `mkdir -p genomics/2_long_reads/nanoplot_raw`
 
 `cd $WORK/genomics/0_raw_reads/long_reads/`
 
 `NanoPlot --fastq $WORK/genomics/0_raw_reads/long_reads/241155E.fastq.gz -o $WORK/genomics/2_long_reads/nanoplot_raw -t 6 --maxlength 40000 --minlength 1000 --plots kde --format png --N50 --dpi 300 --store --raw --tsv_stats --info_in_report`


 - *Filtlong:*
 `mkdir -p genomics/2_long_reads/cleaned_long_reads`
 
 `filtlong --min_length 1000 --keep_percent 90 $WORK/genomics/0_raw_reads/long_reads/*.gz | gzip > $WORK/genomics/2_long_reads/cleaned_long_reads/cleaned_filtlong.fastq.gz`
 `mv cleaned_filtlong.fastq.gz $WORK/genomics/2_long_reads/cleaned_long_reads`


 - *NanoPlot again:*
 `mkdir -p genomics/2_long_reads/cleaned_long_reads`

 `NanoPlot --fastq $WORK/genomics/2_long_reads/cleaned_long_reads/cleaned_filtlong.fastq.gz -o $WORK/genomics/2_long_reads/nanoplot_again_cleaned -t 6 --maxlength 40000 --minlength 1000 --plots kde --format png --N50 --dpi 300 --store --raw --tsv_stats --info_in_report`

 **Questions**
 * *How Good is the long reads quality?*
   -> **before:**
   Reads >Q10: 	11186 (70.1%) 102.2Mb
   Reads >Q15: 	1578 (9.9%) 12.8Mb
   Reads >Q20: 	1 (0.0%) 0.0Mb
   Reads >Q25: 	0 (0.0%) 0.0Mb
   Reads >Q30: 	0 (0.0%) 0.0Mb
 
   n50 	21971.0
   mean_qual 	10.4
   median_qual 	11.7

   -> **after:**
   Reads >Q10: 	10521 (84.5%) 101.4Mb
   Reads >Q15: 	1527 (12.3%) 12.7Mb
   Reads >Q20: 	1 (0.0%) 0.0Mb
   Reads >Q25: 	0 (0.0%) 0.0Mb
   Reads >Q30: 	0 (0.0%) 0.0Mb

   n50 	22747.0
   mean_qual 	11.4
   median_qual 	12.5

 * *How many reads before trimming and how many do you have now?*
   -> **before:** number_of_reads 	15963

   -> **after:** number_of_reads 	12446

3. <Assemble the genome using **Unicycler**>

-> `unicycler -1 1_short_reads_qc/cleaned_241155E_R1.fastq.gz -2 1_short_reads_qc/cleaned_241155E_R2.fastq.gz -l 2_long_reads/cleaned_long_reads/cleaned_filtlong.fastq.gz -o genome_assembly -t 8`

4. **Check the assembly quality**
 1. <Quast>

 -> `mkdir -p $WORK/genomics/genome_assembly/quast`
 
 `quast.py $WORK/genomics/genome_assembly/assembly.fasta --circos -L --conserved-genes-finding --rna-finding --glimmer --use-all-alignments --report-all-metrics -o $WORK/genomics/genome_assembly/quast -t 8`

 -> quast relise on a database BUSCO, thats why we don´t get so many information
 -> we get 6 contigs
 -> but almost the entire assembly is based on one contig
 -> N50 woul be 1, N90 would be 1 propably as well -> because one contig ist sooooo long

 2. <CheckM>
 `mkdir -p $WORK/genomics/genome_assembly/checkm_out`
 # Run CheckM for this assembly
 `checkm lineage_wf $WORK/genomics/genome_assembly $WORK/genomics/genome_assembly/checkm_out -x fasta --tab_table --file $WORK/genomics/genome_assembly/checkm_out/checkm_results -r -t 8`
 # Run CheckM QA for this assembly
 `checkm tree_qa $WORK/genomics/genome_assembly/checkm_out`
 `checkm qa $WORK/genomics/genome_assembly/checkm_out/lineage.ms $WORK/genomics/genome_assembly/checkm_out -o 1 > $WORK/genomics/genome_assembly/checkm_out/final_table_01.csv`
 `checkm qa $WORK/genomics/genome_assembly/checkm_out/lineage.ms $WORK/genomics/genome_assembly/checkm_out -o 2 > $WORK/genomics/genome_assembly/checkm_out/final_table_checkm.csv`

 -> BActeroidales is the marker which was used
 -> Completensess 98.88, Contamination: 0.19
 -> high quality genome 
 -> could easly be found on ncbi as a reference genome
 -> N50: 4.33 -> same which we get on quast
 -> requires a lot of commands to get the infos, different to CheckM2

 3. <CheckM2>
 `mkdir -p $WORK/genomics/genome_assembly/checkm_out2`

 `checkm2 predict --threads 1 --input $WORK/genomics/genome_assembly/assembly.fasta --output-directory $WORK/genomics/genome_assembly/checkm_out2`

 -> is just one line -> is 
 -> Neural Network
 -> completeness: 99.98, Contamination: 0.29 -> differnt to CheckM
 -> is much more sensitve than CheckM, gets a better result

 4. <Bandage>

 [Genome_asssembly_vis](/genome_assembly_day6.png) 

 -> **.gfa** what does that mean: its just a fasta file in in a graphic format
 -> everytime you draw a graph it looks different -> but doesn´t change anything, just draws it in a different layout
 -> one small circular DNA (plasmid) in the cell bzw. bacterium, which is shown
 -> just for getting to know every inofrmation f.e. where is my genome etc.
 -> Genebank file is the fasta file as well but just with additional information with f.e. the genus, names, ...
 -> **.gff is the same but just as a table format**

6. <Classifiy the Genomes with **GTDBTK**>
 -> for thte best performence cpu is reduce and ram is uncreased

 `mkdir -p $WORK/genomics/gtdb_classification`
 then run gtdb:
 `gtdbtk classify_wf --cpus 1 --genome_dir $WORK/genomics/genomes_prokka --out_dir $WORK/genomics/gtdb_classification --extension .fna --skip_ani_screen`
 in the graph 

5. <Annotate the Genomes with **Prokka**>

 `prokka $WORK/genomics/genome_assembly/assembly.fasta --outdir $WORK/genomics/genomes_prokka --kingdom Bacteria --addgenes --cpus 32`

 -> just for getting to know every inofrmation f.e. where is my genome etc.
 -> Genebank file is the fasta file as well but just with additional information with f.e. the genus, names, ...
 -> **.gff is the same but just as a table format**

6. <Classifiy the Genomes with **GTDBTK**>
 -> for thte best performence cpu is reduce and ram is uncreased

 `mkdir -p $WORK/genomics/gtdb_classification`
 then run gtdb:
 `gtdbtk classify_wf --cpus 1 --genome_dir $WORK/genomics/genomes_prokka --out_dir $WORK/genomics/gtdb_classification --extension .fna --skip_ani_screen`

 -> classification, what is predictet for our genome and what is the closest genome (looks at the nucleotides and which are the most similar ones) -> here it is **Bacteroides muris**


7. <MultiQC to combine **reports**>
 -> now running MultiQC to combine all the QC reports at once

 `multiqc -d $WORK/genomics/ -o $WORK/genomics/6_multiqc`
 -> but with this command it creates a directory named genomics with the MultiQC in genomics


 -> just a summary of every quality control which we used
 -> quast results, N50, number of contigs, gene were predict, rna, trna, ...





**Questions**
 * *How good is the quality of genome?*
 -> it is of high quality, low contamination

 * *Why did we use Hybrid assembler?*
 -> because we can combine the pros of short and long reads

 * *What is the difference between short and long reads?*
 -> short reads: higher accuracy -> problem with repetetive regions

 -> long reads: better assembling, because less contigs

 * *Did we use Single or Paired end reads? Why?*
 -> short reads were paired. We had forward an backward reads -> they provide information of both ends and they aprove the assemble accurancy

 * *Which classification was assigned to the genome. Is it trust worthy and why?*
 -> Bacteroides muris -> it is trust worthy, high completeness, low contamination, high confidence of the assignment

**DAY7**

1. <Load the required modules>
2. <Download the data>
3. <Create contigs.dbs from .fasta files>
-> everything with the given commands, just changed the directories

4. <Visualize contigs.db>
 -> `anvi-display-contigs-stats $WORK/pangenomics/V_jascida_genomes/*db`
 -> [Visualisation of contigs](/day7task4.png) 
 -> [Visualisation of contigs](/day7task4.2.png) 

5. <Create external genomes file>
 -> `anvi-script-gen-genomes-file --input-dir $WORK/pangenomics/V_jascida_genomes -o pangenomics/external-genomes.txt`

6. <Investigate contamination>
 -> directly in the terminal:`cd $WORK/pangenomics/V_jascida_genomes`
 `anvi-estimate-genome-completeness -e external-genomes.txt`
 -> [Contamination Investigation](/day7task6.png) 

 -> one genome with 21.13 redundancy. Its a sign for contamination. SO we want ot look at this closer -> its the genome 52
 

7. <Visualise contigs for refinement>
 -> and then to identify untypical bins ("bad" bins) 
 `cd V_jascida_genomes`
 `anvi-estimate-genome-completeness -e external-genomes.txt`

 -> visualisation in interactive:
 `anvi-interactive -c pangenomics/V_jascida_genomes/V_jascida_52.db -p pangenomics/V_jascida_genomes/V_jascida_52/PROFILE.db`

 -> **Bild von jemand anderem?, hat bei mir nicht funktioniert**
 -> every line is a contig, they are grouped how likely they are to belonging together
 -> then create a bin and mark what we think is contaminated
 -> antoher bin with clean which shouldn´t have a high Redundancy -> when it is at 5.6 its like evry other genome
 -> then store the bin

8. <Splitting the genome in our good bins>
 -> here we will split the PROFILE database and get rid of the cotaminated bin

9. <Estimate completeness of split vs. unsplit genome>
 -> `anvi-estimate-genome-completeness -e $WORK/pangenomics/V_jascida_genomes/external-genomes.txt -o $WORK/pangenomics/V_jascida_genomes/genome_completeness.txt`


10. <Compute pangenome>
 -> `cd $WORK/pangenomics/V_jascida_genomes`
 # generate a genome storage database
 `anvi-gen-genomes-storage -e external-genomes.txt -o V_jascida-GENOMES.db`

 # calculate the pangenome
 `anvi-pan-genome -g V_jascida-GENOMES.db --project-name V_jascida --num-threads 8`   

 # calculate the ANI 
 `anvi-compute-genome-similarity -e external-genomes.txt -o ani -p V_jascida/V_jascida-PAN.db -T 8`  

 -> but not working with the external final textfile because step 7 and 8 did not really work                         


11. <Display the pangenome>
 -> visulisation in interactive in terminal
 -> only difference to the pictures of the others is that in mine the contamination isn´t removed
 -> in labels, click ANI_percentage_identity




12 <Computing Phylogenomics for your pangenome (Optional task)>


**Questions**
 * *Are genes clustered base on sequence similarity or functional annotation?*
 -> based on sequence similarity -> it compares every detail to see how likely they are

 * *How do you spot a "bad" genome, or "bad" bin in a genome?*
 -> I can spot it when I look at the redundancy, because a high redundancy is a hint of a contamination. When I want to find a bad bin I can just visualize the genome and spot it then 
 -> as well low completeness

 * *Use the search function to assign all gene clusters into the following bins: Core genome, Accessory genome, Singletons and Single Copy core genes (SCGs). Include a screenshot of your pangenome into the protocol*
 -> [Pangenomic](/day7Questions.png) 

 * *If you add more genomes to the pangenome, what would happen to the number of gene clusters in the Core genome and in SCGs.*
 -> the core genome decrease. Feweer genes are shared by all genomes
 -> number of SCGs also decrease -> because then some genes could maybe no longer represented in every genome or appear in multiple copies

 * *Based on the ANI, would you say all genomes belong to the same species?*
 -> If its an Average Nucleotide Identity (ANI) same or higher than 95% than all the genomes are likely to belong to the same species


 -> redundancy: hinweis auf contamination, kann aber auch sein, dass ein Gen einfach doppelt vorliegt


**DAY8**
* Aim in the tutorial: Setting up an RNA-Seq data analysis pipeline; RNA-Seq data pre-processing; RNA-Seq data analysis; Differential gene expression analysis; Data visualization
* What we do: Download the required files; Set up the files in specific locations; Check read quality; Align the reads to a reference sequence; Calculate the coverage; Perform gene wise quantification; Calculate differential gene expression

* **<Example 1>**
-> complete script to run an RNA-Seq analysis using READemption is given

-> InSPI2 describes an acidic phosphate-limiting minimal medium that induces Salmonella pathogenicity island (SPI) 2 transcription. This is suspected to create an environmental shock that could induce the upregulation of specific gene sets.
-> LSP (late stationary phase). This is reached by growth in Lennox Broth medium.

-> **Explained:**
-> **Transicrptomics is about comparing things**: in Example 1 we are comparing InSPI2 and LSP 
-> `export` to access the internet with proxy environment, because you can´t access directly
-> creating folders for READemption with projectname
-> get the reference genome (in this case 1 genome and 3 plasmids) and put them in specific folders
-> then renaming the files (not always necassary)
-> reference alone doesn´t help that much, you haveto understand it -> download the annotation as well -> and in this case unzip it
-> two conditions R1 and R2 as well
-> now there is everything to start
-> let READemption work and let it visualize it



* **<Example 2>**

1. Download the sequence data you want to analyze
-> searching in the linkied article for the accession number **GSE85456**
-> on NCBI: going Genomes -> BioProject -> Project Data: SRA Experiments -> finding four links with each of it having the SSR number on the end of the website

-> SRX2012145: GSM2267309: ∆sRNA154 replicate 1; Methanosarcina mazei Go1; RNA-Seq with: `SRR4018516` (6.0M(Spots)	598.9M(Bases)	380.6MB(Size)	39.5%(GC Conetent))

-> SRX2012144: GSM2267308: wildtype replicate 2; Methanosarcina mazei Go1; RNA-Seq with: `SRR4018515` (4.8M	477.6M	304.7MB	40.3%)

-> SRX2012143: GSM2267307: wildtype replicate 1; Methanosarcina mazei Go1; RNA-Seq with: `SRR4018514` (3.8M	376.8M	241.0MB	41.5%)

-> SRX2012146: GSM2267310: ∆sRNA154 replicate 2; Methanosarcina mazei Go1; RNA-Seq with: `SRR4018517` (6.8M	682.4M	432.5MB	39.8%)

-> SRR numbers are the actual reads themself: das sind also eindeutige „Run Accession Numbers“ im Sequence Read Archive (SRA) – also Identifikationsnummern für einzelne Sequenzier-Runs
**-> running everthing in the terminal except step 5**
in teriminal:
```
grabseqs sra -t 4 -m ./metadata.csv SRR4018516
grabseqs sra -t 4 -m ./metadata.csv SRR4018515
grabseqs sra -t 4 -m ./metadata.csv SRR4018514
grabseqs sra -t 4 -m ./metadata.csv SRR4018517
```
-> and then renaming it

2. Create the reademption folder structure

`reademption create --project_path READemption_analysis_2 --species methanosarcina="Methanosarcina mazei Gö1"`

3. Download stuff

# download reference genome
wget -O READemption_analysis_2/input/methanosarcina_reference_sequences/GCF_000007065.1_ASM706v1_genomic.fna.gz https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/007/065/GCF_000007065.1_ASM706v1/GCF_000007065.1_ASM706v1_genomic.fna.gz

# download annotation
wget -O READemption_analysis_2/input/methanosarcina_annotations/GCF_000007065.1_ASM706v1_genomic.gff.gz https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/007/065/GCF_000007065.1_ASM706v1/GCF_000007065.1_ASM706v1_genomic.gff.gz

# unzip them
gunzip READemption_analysis_2/input/methanosarcina_reference_sequences/GCF_000007065.1_ASM706v1_genomic.fna.gz
gunzip READemption_analysis_2/input/methanosarcina_annotations/GCF_000007065.1_ASM706v1_genomic.gff.gz

4. Copy the raw_reads to the READemption_analysis folder

cp *.fastq.gz reademption_folder/READemption_analysis_2/input/reads

5. Run READemption

# align reads to reference
reademption align -p 4 --poly_a_clipping --project_path READemption_analysis_2 --fastq

# calculate read coverage
reademption coverage -p 4 --project_path READemption_analysis_2

# quantify gene expression
reademption gene_quanti -p 4 --features CDS,tRNA,rRNA --project_path READemption_analysis_2

# calculate differential expression using DESeq2
reademption deseq -l mut_R1.fastq.gz,mut_R2.fastq.gz,wt_R1.fastq.gz,wt_R2.fastq.gz -c mut,mut,wt,wt -r 1,2,1,2 --libs_by_species methanosarcina=mut_R1,mut_R2,wt_R1,wt_R2 --project_path READemption_analysis_2

# visualization
reademption viz_align --project_path READemption_analysis_2
reademption viz_gene_quanti --project_path READemption_analysis_2
reademption viz_deseq --project_path READemption_analysis_2


6. Analyze your results

 * all_species_viz_align:
   -> it is a box plot which shows that all reads could be assigned to the refernce genome `Methanosarcina mazei`, which is expectet. There were just some unaligned reads. We don´t have cross aligned reads, because they were aligned to mulitple genomes and we only tested for one.


 * methanosarcina_viz_align:
   -> it is a box plot. We can see that there are many uniquely aligned reads and just some multiple aligned reads of it. 
   Multiple means they are reads that cover more genes bzw. go over one gene.
   split aligend reads means if we had two genomes they were aligned on both but not together, but we don´t have that.


 * methanosarcina_viz_deseq:
 -> MA plot:
   -> it shows dots in red and black, while the every dot stands for one gene. The red dots are significantly differentially expressed genes. Black dots arent significantly and with a unsignficant p-value.
   It shows that many dots are on the left side in the middel which means they are    expressed low
   -> but no expression difference between left and right side
   -> both plots look the same because they were just flipped its mut vs. wt and wt vs. mut.
   -> y-Achse with log2 fold: 0 means no change, 1 means douuble the expression, -1 means that its half of that expression

 -> volcano plot:
   -> just a different way of representation of the data in MA plot 
   -> on the right side they are high regulated on the left low regulated. If they are down, near 0 on the y-Achse, its not signigicant .
   -> so we can say that both plots arent that sgnifikant with a low p-value
  

 * methanosarciana_viz_gene_quanti:
 -> expression scatter plot:
   -> more like a quality control
   -> used to compare the expression of every gene in different conditions. Every dot represents one gene. THey are mostly aligned around the diagonal line. If they are directly on the line it says that they have the exact same expression in both condtions and when they are near the line its barley a differnece
   -> but we want some difference, but not completely different

 -> RNA class box plots:
   -> shows distributionn of the geneexpressionvalues its usually used for quality control and comparing of the samples. 
   -> actual conding sequence is shown

 * read_lengths_viz_align:
   -> read length distribution plots compares it before and after trimming. Before they were 100 nucelotides long and afterwards, after trimming, they got a bite shorter. 




7. What now?
 -> most important files in methanosarcina_deseq with tables that show the results of the differntial expression analysis (log2 FC, p-values etc.); **Think about what you could do with these results.**

 -> contains a lot of information:
   -> contains information of reads 
   -> reademption uses multiple sources for annotations
   -> GO trams(?) = trams in which are genes are grouped into certain categories
   -> raw counts
   -> most important: log2fold change and the p-value (which is a measure of the significance) nd adjustet p-values (which is just a bit stricter)
     -> mostly with a signifcance value of 0.05 -> everthing lower means that it is signficant  

   -> it contains so much that you can do a lot if it, just filter the table for what you want to do or test for :)
   

-> **Bilder einfügen**



**DAY9** <Viromics>

1. <Aim>
-> learn about virome analysis for a metagenomics dataset, which include these steps:  
  Read pre-processing and assembly (either virus-specific assembly or standard metagenomic assembly)
  Viral identification
  Virus-specific quality control
   Viral clustering
  Viral binning
  Abundance estimation
  Virus-specific annotation
  Host prediction
and maybe as well as additional steps:  
  Viral taxonomy prediction
  Viral Lifecycle prediction
  Virus-specific metabolic analyses

2. <Tools and pipelines>
  * MVP (Modular Viromics Pipeline)
    -> follows a robust workflow
    -> outputs are structured really well 
    -> uses clean reads and assemblies from multiple samples 
  * iPHoP (integrated Phage-Host Prediction)
    -> Since MVP only prepares output for host prediction via iPHoP (some tools need special formats to work) but does not run the tool itself, we will be manually running iPHoP

3. <Commands>


4. <Questions> 
  1. 
   `grep ">" -c 01_GENOMAD/BGR_140717/BGR_140717_Viruses_Genomad_Output/BGR_140717_modified_summary/*_virus.fna`
   846
  2. 
   `grep ">" -c 01_GENOMAD/BGR_140717/BGR_140717_Proviruses_Genomad_Output/proviruses_summary/*_virus.fna`
   11
  3. 
    `grep -c "Caudoviricetes" 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv`
   02_CHECK_V/BGR_130305/MVP_02_BGR_130305_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:614
   02_CHECK_V/BGR_130527/MVP_02_BGR_130527_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:684
   02_CHECK_V/BGR_130708/MVP_02_BGR_130708_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:850
   02_CHECK_V/BGR_130829/MVP_02_BGR_130829_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:954
   02_CHECK_V/BGR_130925/MVP_02_BGR_130925_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:849
   02_CHECK_V/BGR_131021/MVP_02_BGR_131021_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:1079
   02_CHECK_V/BGR_131118/MVP_02_BGR_131118_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:609
   02_CHECK_V/BGR_140106/MVP_02_BGR_140106_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:368
   02_CHECK_V/BGR_140121/MVP_02_BGR_140121_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:575
   02_CHECK_V/BGR_140221/MVP_02_BGR_140221_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:717
   02_CHECK_V/BGR_140320/MVP_02_BGR_140320_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:468
   02_CHECK_V/BGR_140423/MVP_02_BGR_140423_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:358
   02_CHECK_V/BGR_140605/MVP_02_BGR_140605_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:527
   02_CHECK_V/BGR_140717/MVP_02_BGR_140717_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:559
   02_CHECK_V/BGR_140821/MVP_02_BGR_140821_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:337
   02_CHECK_V/BGR_140919/MVP_02_BGR_140919_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:476
   02_CHECK_V/BGR_141022/MVP_02_BGR_141022_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:401
   02_CHECK_V/BGR_150108/MVP_02_BGR_150108_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:339

  4. 
    `grep -c "Unclassified" 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv`
   02_CHECK_V/BGR_130305/MVP_02_BGR_130305_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:10
   02_CHECK_V/BGR_130527/MVP_02_BGR_130527_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:9
   02_CHECK_V/BGR_130708/MVP_02_BGR_130708_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:10
   02_CHECK_V/BGR_130829/MVP_02_BGR_130829_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:14
   02_CHECK_V/BGR_130925/MVP_02_BGR_130925_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:11
   02_CHECK_V/BGR_131021/MVP_02_BGR_131021_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:14
   02_CHECK_V/BGR_131118/MVP_02_BGR_131118_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:7
   02_CHECK_V/BGR_140106/MVP_02_BGR_140106_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:4
   02_CHECK_V/BGR_140121/MVP_02_BGR_140121_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:4
   02_CHECK_V/BGR_140221/MVP_02_BGR_140221_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:8
   02_CHECK_V/BGR_140320/MVP_02_BGR_140320_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:4
   02_CHECK_V/BGR_140423/MVP_02_BGR_140423_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:4
   02_CHECK_V/BGR_140605/MVP_02_BGR_140605_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:7
   02_CHECK_V/BGR_140717/MVP_02_BGR_140717_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:9
   02_CHECK_V/BGR_140821/MVP_02_BGR_140821_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:4
   02_CHECK_V/BGR_140919/MVP_02_BGR_140919_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:8
   02_CHECK_V/BGR_141022/MVP_02_BGR_141022_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:8
   02_CHECK_V/BGR_150108/MVP_02_BGR_150108_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:10

  5. -> **Tabelle machen!**
   `grep -v "Unclassified" 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv |grep -v "Caudoviricetes" | grep -v "Sample"`
   02_CHECK_V/BGR_130305/MVP_02_BGR_130305_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:BGR_130305	BGR_130305_NODE_2265_length_4061_cov_5.941338	4061	No	3	1	0	Low-quality	Genome-fragment	2.89	AAI-based (medium-confidence)	1.0		11	0.9643	04.4395	Viruses;Varidnaviria;Bamfordvirae;Nucleocytoviricota;Megaviricetes;Imitervirales;Mimiviridae	dsDNA	Eukaryote
   02_CHECK_V/BGR_130527/MVP_02_BGR_130527_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:BGR_130527	BGR_130527_NODE_3972_length_2853_cov_3.214796	2853	No	3	1	0	Low-quality	Genome-fragment	4.0	HMM-based (lower-bound)	1.0		11	0.8411	0	3.4351	Viruses;Varidnaviria;Bamfordvirae;Nucleocytoviricota;Megaviricetes;Algavirales;Phycodnaviridae	dsDNA	Eukaryote
   02_CHECK_V/BGR_130708/MVP_02_BGR_130708_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:BGR_130708	BGR_130708_NODE_5125_length_2609_cov_6.036805	2609	No	3	1	0	Low-quality	Genome-fragment	3.65	HMM-based (lower-bound)	1.0		11	0.7483	0	3.4351	Viruses;Varidnaviria;Bamfordvirae;Nucleocytoviricota;Megaviricetes;Algavirales;Phycodnaviridae	dsDNA	Eukaryote
   02_CHECK_V/BGR_131021/MVP_02_BGR_131021_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:BGR_131021	BGR_131021_NODE_3917_length_3744_cov_3.725400	3744	No	4	1	0	Low-quality	Genome-fragment	1.08	AAI-based (medium-confidence)	1.0		11	0.9654	12.6771	Viruses	Unknown	Unknown
   02_CHECK_V/BGR_131021/MVP_02_BGR_131021_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:BGR_131021	BGR_131021_k141_151878_length_2551_cov_25.4033	2551	No	5	1	0	Low-quality	Genome-fragment	6.07	HMM-based (lower-bound)	1.0		11	0.9284	0	1.7183	Viruses;Varidnaviria;Bamfordvirae;Preplasmiviricota;Tectiliviricetes;Autolykiviridae	dsDNA	Prokaryote
   02_CHECK_V/BGR_131021/MVP_02_BGR_131021_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:BGR_131021	BGR_131021_NODE_12517_length_1678_cov_37.389402	1678	No	3	1	0	Low-quality	Genome-fragment	16.44	HMM-based (lower-bound)	1.0		11	0.9323	1	1.7183	Viruses;Riboviria;Pararnavirae;Artverviricota;Revtraviricetes;Ortervirales;Retroviridae	ssRNA-RT	Eukaryote
   02_CHECK_V/BGR_140221/MVP_02_BGR_140221_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:BGR_140221	BGR_140221_NODE_2074_length_4089_cov_7.783837_1_1_3696	3696	Yes	4	1	0	Low-quality	Genome-fragment	3.02	AAI-based (medium-confidence)	1.0	1-3696	11	0.9072	0	1.7183	Viruses;Varidnaviria;Bamfordvirae;Nucleocytoviricota;Megaviricetes;Algavirales;Phycodnaviridae	dsDNA	Eukaryote
   02_CHECK_V/BGR_140605/MVP_02_BGR_140605_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:BGR_140605	BGR_140605_NODE_5969_length_1911_cov_2.155172	1911	No	5	3	0	Low-quality	Genome-fragment	39.82	AAI-based (medium-confidence)	1.0		11	0.97	14.5511	Viruses;Monodnaviria;Sangervirae;Phixviricota;Malgrandaviricetes;Petitvirales;Microviridae	ssDNA	Prokaryote
   02_CHECK_V/BGR_140919/MVP_02_BGR_140919_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:BGR_140919	BGR_140919_NODE_1812_length_5087_cov_5.712838	5087	No	10	1	1	Low-quality	Genome-fragment	0.41	HMM-based (lower-bound)	1.0		11	0.9463	0	4.0154	Viruses	Unknown	Unknown
   02_CHECK_V/BGR_140919/MVP_02_BGR_140919_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:BGR_140919	BGR_140919_k141_2996_length_1161_cov_5.0000	1161	No	2	1	0	Low-quality	Genome-fragment	17.74	HMM-based (lower-bound)	1.0		11	0.959	1	1.5528	Viruses;Monodnaviria;Sangervirae;Phixviricota;Malgrandaviricetes;Petitvirales;Microviridae	ssDNA	Prokaryote

  6. 
    `cut` to cut the columns that are wanted
    `-f ?` to select a specific coloumn, in this case the check_quality (which is Low-quality or High_quality)
    `grep` for the rows
    `-e` to search for a specific pattern or more like the "High-quality" and "Complete"
    `-c` to count every columns that aren´t empty
   `cut -f 8 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -e "High-quality" -e "Complete" |grep -c ""`
   -> 5 
   -> without `grep -c""` it lists it: We have 3x Complete and 2x High-quality
  
  7. 
   -> 5x loops and just switch the grep -c "" for Low-quality, Medium-quality, High-quality, Complete and Sample -> or just do all in one with echo(?)
   -> `break` to end the loop aftre the first file, just to check if the command worked the like it should 

   `for x in BGR_130305  BGR_130527  BGR_130708  BGR_130829  BGR_130925  BGR_131021  BGR_131118  BGR_140106  BGR_140121  BGR_140221  BGR_140320  BGR_140423  BGR_140605  BGR_140717  BGR_140821  BGR_140919  BGR_141022  BGR_150108; do echo $x; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "Low-quality" ;break; done` -> with break to test

   `for x in BGR_130305  BGR_130527  BGR_130708  BGR_130829  BGR_130925  BGR_131021  BGR_131118  BGR_140106  BGR_140121  BGR_140221  BGR_140320  BGR_140423  BGR_140605  BGR_140717  BGR_140821  BGR_140919  BGR_141022  BGR_150108; do echo $x; echo  "Low-quality"; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "Low-quality" ; echo  "Medium-quality"; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "Medium-quality"; echo  "High-quality"; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "High-quality"; echo  "Complete"; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "Complete"; done`
   BGR_130305
   Low-quality 625
   Medium-quality 0
   High-quality 0
   Complete 0

   BGR_130527
   Low-quality 693
   Medium-quality 1
   High-quality 0
   Complete 0

   BGR_130708
   Low-quality 860
   Medium-quality 1
   High-quality 0
   Complete 0

   BGR_130829
   Low-quality 964
   Medium-quality 3
   High-quality 0
   Complete 0

   BGR_130925
   Low-quality 856
   Medium-quality 3
   High-quality 0
   Complete 0
   
   BGR_131021
   Low-quality 1091
   Medium-quality 3
   High-quality 0
   Complete 1
   
   BGR_131118
   Low-quality 613
   Medium-quality 3
   High-quality 0
   Complete 0
   
   BGR_140106
   Low-quality 372
   Medium-quality 0
   High-quality 0
   Complete 0
   
   BGR_140121
   Low-quality 578     
   Medium-quality 0
   High-quality 0
   Complete 1
   
   BGR_140221
   Low-quality 724
   Medium-quality 1
   High-quality 1
   Complete 0
   
   BGR_140320
   Low-quality 471
   Medium-quality 1
   High-quality 0
   Complete 0
   
   BGR_140423
   Low-quality 362
   Medium-quality 0
   High-quality 0
   Complete 0
   
   BGR_140605
   Low-quality 531
   Medium-quality 2
   High-quality 0
   Complete 0
   
   BGR_140717
   Low-quality 564
   Medium-quality 3
   High-quality 0
   Complete 1
  
   BGR_140821
   Low-quality 338
   Medium-quality 1
   High-quality 1
   Complete 0
   
   BGR_140919
   Low-quality 485
   Medium-quality 1
   High-quality 0
   Complete 0
   
   BGR_141022
   Low-quality 409
   Medium-quality 0
   High-quality 0
   Complete 0
   
   BGR_150108
   Low-quality 348
   Medium-quality 1
   High-quality 0
   Complete 0

  8. 
  `grep "Complete" 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv > 8.csv`
   `> 8.csv` -> to create a textfile with the solution


  9. 
    * In what samples were the complete viruses found?
     -> in BGR_131021, BGR_140121 and BGR_140717
    * Are they integrated proviruses?
     -> nope
    * How long are they?
     -> BGR_131021: 46113
     -> BGR_140121: 34619
     -> BGR_140717: 31258 
    * How many viral hallmark genes do they have?
     -> BGR_131021: 6
     -> BGR_140121: 10
     -> BGR_140717: 10 
    * What percentage of the viral genes are viral hallmark genes? You may round to full numbers.
     -> BGR_131021: 12 viral genes so 50% viral hallmark genes
     -> BGR_140121: 24 viral genes so 42% viral hallmark genes
     -> BGR_140717: 21 viral genes so 48% viral hallmark genes
    * Thinking for yourself: Why are there more genes (gene_count) than viral genes and host genes combined? Tip: What does the column gene_count tell us?
     -> gene_count tell us the total number of genes predicted on it
     -> so there are more genes than viral genes and host genes combined because many genes are unclassified so they don´t count as viral genes or viral hallmark genes

  10. Clustering Output: difference between the three tsv files
    -> there is unfiltered, filtered and filtered + representative. In the unfiltered file is everthing. In the filtered file is everthing, that wasn´t filtered out, because it wasn´t a virus like thin. The filtered and representative file just have one representative bin there for each.

  11. 
 `grep -c "" 03_CLUSTERING/MVP_03_All_Sample_Filtered_Relaxed_Merged_Genomad_CheckV_Representative_Virus_Proviruses_Quality_Summary.tsv`
   -> 5375 cluster representatives

  12. 
 `cut -f 5 03_CLUSTERING/MVP_03_All_Sample_Filtered_Relaxed_Merged_Genomad_CheckV_Representative_Virus_Proviruses_Quality_Summary.tsv | grep -c "Yes"`
   -> `grep -c "Yes"` because we just wanted to count the ones that were actually with proviruses
   -> 91 are proviruses

  13. What clusters do the complete viruses from 8) belong to? How large are the clusters?
   -> unique virus_id: 
   BGR_131021_NODE_96_length_46113_cov_32.412567
   BGR_140121_NODE_54_length_34619_cov_66.823718
   BGR_140717_NODE_168_length_31258_cov_37.020094
   `grep -e "BGR_131021_NODE_96_length_46113_cov_32.412567" -e "BGR_140121_NODE_54_length_34619_cov_66.823718" -e "BGR_140717_NODE_168_length_31258_cov_37.020094" 03_CLUSTERING/MVP_03_All_Sample_Filtered_Relaxed_Merged_Genomad_CheckV_Representative_Virus_Proviruses_Quality_Summary.tsv`


   BGR_131021_NODE_96_length_46113_cov_32.412567	BGR_131021_NODE_96_length_46113_cov_32.412567,BGR_130829_NODE_285_length_18697_cov_26.889014,BGR_130925_NODE_229_length_19502_cov_14.312747,BGR_130925_NODE_141_length_27707_cov_13.801208,BGR_130829_NODE_713_length_9520_cov_30.608875,BGR_130829_NODE_699_length_9615_cov_30.338912,BGR_130829_NODE_324_length_17453_cov_27.959938	BGR_131021	46113	No	53	12	2	Complete	High-quality	100.0DTR (high-confidence)	1.0		11	0.9741	6	26.0838	Viruses;Duplodnaviria;Heunggongvirae;Uroviricota;Caudoviricetes	dsDNA	Prokaryote
    -> **7 Virus IDs**


   BGR_140121_NODE_54_length_34619_cov_66.823718	BGR_140121_NODE_54_length_34619_cov_66.823718,BGR_140221_NODE_108_length_34115_cov_16.844598,BGR_140320_NODE_81_length_29543_cov_29.496609,BGR_140821_NODE_2390_length_3137_cov_7.984101,BGR_141022_NODE_3686_length_2801_cov_14.556082,BGR_140320_NODE_2744_length_2659_cov_15.275730,BGR_130305_NODE_4045_length_2973_cov_59.104181,BGR_141022_NODE_4172_length_2586_cov_16.268669,BGR_150108_NODE_3086_length_2780_cov_10.336514,BGR_130527_NODE_4596_length_2616_cov_14.925029,BGR_130925_NODE_3974_length_2359_cov_11.524740,BGR_140919_NODE_5494_length_2186_cov_12.972783,BGR_141022_NODE_5999_length_2061_cov_31.139083,BGR_131021_NODE_10677_length_1861_cov_7.460133,BGR_140821_NODE_6440_length_1569_cov_3.940555,BGR_140320_NODE_3232_length_2377_cov_5.028854,BGR_150108_NODE_6896_length_1666_cov_38.685909,BGR_130925_NODE_8017_length_1619_cov_18.003836,BGR_140821_k141_52915_length_1214_cov_8.0000,BGR_131021_NODE_6902_length_2517_cov_5.516653,BGR_130829_k141_4358_length_1184_cov_20.0144,BGR_130925_k141_23197_length_1060_cov_20.8466,BGR_130829_NODE_12350_length_1519_cov_13.954918,BGR_130305_k141_6566_length_1327_cov_14.0000,BGR_150108_k141_43350_length_1063_cov_24.0000	BGR_140121	34619	No	47	24	0	Complete	High-quality	100.0	DTR (high-confidence)1.0		11	0.9812	10	41.8446	Viruses;Duplodnaviria;Heunggongvirae;Uroviricota;Caudoviricetes	dsDNA	Prokaryote
    -> **25 Virus IDs**


   BGR_140717_NODE_168_length_31258_cov_37.020094	BGR_140717_NODE_168_length_31258_cov_37.020094,BGR_130829_NODE_4298_length_2731_cov_17.951420,BGR_130925_NODE_3352_length_2611_cov_9.501174,BGR_140919_NODE_6367_length_1998_cov_27.025219,BGR_130829_NODE_11382_length_1587_cov_17.503916,BGR_140821_NODE_6386_length_1578_cov_11.059750,BGR_141022_NODE_9267_length_1584_cov_17.466318,BGR_141022_NODE_8692_length_1646_cov_8.513514,BGR_130305_k141_4034_length_1514_cov_24.1129,BGR_130708_k141_82351_length_1001_cov_17.0000,BGR_130708_k141_146703_length_1144_cov_13.0419	BGR_140717	31258	No	44	21	0	Complete	High-quality	100.0	DTR (medium-confidence)	1.0		11	0.9818	10	40.2292	Viruses;Duplodnaviria;Heunggongvirae;Uroviricota;Caudoviricetes	dsDNA	Prokaryote
    -> **11 Virus IDs**


  14. Thinking for yourself: Now we want to look at abundances. The files can be found in 04_READ_MAPPING, split per sample. Open any of the CoverM files and ignore everything except the first two columns. What does this file tell you, conceptionally? Meaning: What do the lines in this file represent (i.e. what kind of data is described in line 2, line 3, .....), who do the IDs listed in the second column belong to, what kind of data was combined to generate this file, and what can we learn from it? 
   -> every CoverM file contains a sample which is mapped against all the viral cluster representatives.


  15. Thinking for yourself: If you scroll through the CoverM file you selected, you will come across lines where all of the metrics (except length) are zero. How could this have happened? Is it a bug or a feature?!
   -> These zero value are there because no reads from this sample are mapped to the viral references

  16. Merge all CoverM files for all the samples (order doesn't matter) using e.g. the cat command (not cut). Remember to exclude the headers and keep only one. Your finished file should have 96733 lines total.
   -> `grep --no-filename -v "Sample" 04_READ_MAPPING/BGR_*/*_CoverM.tsv >04_READ_MAPPING/all_coverm_temp.tsv ; grep "Sample" 04_READ_MAPPING/BGR_150108/BGR_150108_CoverM.tsv >04_READ_MAPPING/temp_tmp.tsv ; cat 04_READ_MAPPING/temp_tmp.tsv 04_READ_MAPPING/all_coverm_temp.tsv >04_READ_MAPPING/all_coverm.tsv`
   -> change path: cd $WORK/viromics/04_READ_MAPPING
   -> `grep -c "" all_coverm.tsv`
   -> 96733


  17. Using the file you just created, how abundant are the complete viruses in different samples (RPKM)? Create a table.
   -> `grep -e "BGR_131021_NODE_96_length_46113_cov_32.412567" -e "BGR_140121_NODE_54_length_34619_cov_66.823718" -e "BGR_140717_NODE_168_length_31258_cov_37.020094" 04_READ_MAPPING/all_coverm.tsv; for x in "BGR_131021_NODE_96_length_46113_cov_32.412567" "BGR_140121_NODE_54_length_34619_cov_66.823718"  "BGR_140717_NODE_168_length_31258_cov_37.020094" ; do grep "$x" 04_READ_MAPPING/all_coverm.tsv | cut -f1,2,11 ; done > EVERTHING_EVERYWHERE_ALL_AT_ONCE.tsv`

   -> output from EVERTHING_EVERYWHERE_ALL_AT_ONCE.tsv
   BGR_130305	BGR_131021_NODE_96_length_46113_cov_32.412567	0.0
   BGR_130527	BGR_131021_NODE_96_length_46113_cov_32.412567	0.0
   BGR_130708	BGR_131021_NODE_96_length_46113_cov_32.412567	10.156808
   BGR_130829	BGR_131021_NODE_96_length_46113_cov_32.412567	201.0595
   BGR_130925	BGR_131021_NODE_96_length_46113_cov_32.412567	189.07722
   BGR_131021	BGR_131021_NODE_96_length_46113_cov_32.412567	154.71898
   BGR_131118	BGR_131021_NODE_96_length_46113_cov_32.412567	11.161729
   BGR_140106	BGR_131021_NODE_96_length_46113_cov_32.412567	31.22893
   BGR_140121	BGR_131021_NODE_96_length_46113_cov_32.412567	17.252619
   BGR_140221	BGR_131021_NODE_96_length_46113_cov_32.412567	0.60999864
   BGR_140320	BGR_131021_NODE_96_length_46113_cov_32.412567	0.0
   BGR_140423	BGR_131021_NODE_96_length_46113_cov_32.412567	13.359601
   BGR_140605	BGR_131021_NODE_96_length_46113_cov_32.412567	9.183679
   BGR_140717	BGR_131021_NODE_96_length_46113_cov_32.412567	0.12582351
   BGR_140821	BGR_131021_NODE_96_length_46113_cov_32.412567	0.099048875
   BGR_140919	BGR_131021_NODE_96_length_46113_cov_32.412567	0.0
   BGR_141022	BGR_131021_NODE_96_length_46113_cov_32.412567	0.0
   BGR_150108	BGR_131021_NODE_96_length_46113_cov_32.412567	2.3968215
   BGR_130305	BGR_140121_NODE_54_length_34619_cov_66.823718	530.0747
   BGR_130527	BGR_140121_NODE_54_length_34619_cov_66.823718	345.76505
   BGR_130708	BGR_140121_NODE_54_length_34619_cov_66.823718	199.11523
   BGR_130829	BGR_140121_NODE_54_length_34619_cov_66.823718	75.662476
   BGR_130925	BGR_140121_NODE_54_length_34619_cov_66.823718	88.48908
   BGR_131021	BGR_140121_NODE_54_length_34619_cov_66.823718	11.8905115
   BGR_131118	BGR_140121_NODE_54_length_34619_cov_66.823718	2.9122066
   BGR_140106	BGR_140121_NODE_54_length_34619_cov_66.823718	0.46738622
   BGR_140121	BGR_140121_NODE_54_length_34619_cov_66.823718	1329.697
   BGR_140221	BGR_140121_NODE_54_length_34619_cov_66.823718	210.0382
   BGR_140320	BGR_140121_NODE_54_length_34619_cov_66.823718	472.9365
   BGR_140423	BGR_140121_NODE_54_length_34619_cov_66.823718	16.567926
   BGR_140605	BGR_140121_NODE_54_length_34619_cov_66.823718	3.1911626
   BGR_140717	BGR_140121_NODE_54_length_34619_cov_66.823718	72.168
   BGR_140821	BGR_140121_NODE_54_length_34619_cov_66.823718	100.86392
   BGR_140919	BGR_140121_NODE_54_length_34619_cov_66.823718	152.32184
   BGR_141022	BGR_140121_NODE_54_length_34619_cov_66.823718	200.0772
   BGR_150108	BGR_140121_NODE_54_length_34619_cov_66.823718	267.48816
   BGR_130305	BGR_140717_NODE_168_length_31258_cov_37.020094	197.46283
   BGR_130527	BGR_140717_NODE_168_length_31258_cov_37.020094	161.75777
   BGR_130708	BGR_140717_NODE_168_length_31258_cov_37.020094	113.52365
   BGR_130829	BGR_140717_NODE_168_length_31258_cov_37.020094	43.803528
   BGR_130925	BGR_140717_NODE_168_length_31258_cov_37.020094	46.017822
   BGR_131021	BGR_140717_NODE_168_length_31258_cov_37.020094	6.9410787
   BGR_131118	BGR_140717_NODE_168_length_31258_cov_37.020094	3.3950949
   BGR_140106	BGR_140717_NODE_168_length_31258_cov_37.020094	3.1058502
   BGR_140121	BGR_140717_NODE_168_length_31258_cov_37.020094	292.06485
   BGR_140221	BGR_140717_NODE_168_length_31258_cov_37.020094	46.794453
   BGR_140320	BGR_140717_NODE_168_length_31258_cov_37.020094	153.9964
   BGR_140423	BGR_140717_NODE_168_length_31258_cov_37.020094	3.3980339
   BGR_140605	BGR_140717_NODE_168_length_31258_cov_37.020094	2.0616696
   BGR_140717	BGR_140717_NODE_168_length_31258_cov_37.020094	305.86407
   BGR_140821	BGR_140717_NODE_168_length_31258_cov_37.020094	85.26142
   BGR_140919	BGR_140717_NODE_168_length_31258_cov_37.020094	130.4371
   BGR_141022	BGR_140717_NODE_168_length_31258_cov_37.020094	154.94527
   BGR_150108	BGR_140717_NODE_168_length_31258_cov_37.020094	194.18689


**Annotation**
  18. 
 The .tsv files show gene annotations for the viral cluster representatives, with one using conservative and the other relaxed filtering (conservative has fewer lines). The .faa files contain the predicted protein sequences for cluster representatives and all viruses.

  19. We will focus on the file with the conservative results. Find the lines regarding the complete viruses.

  How many genes does the complete virus have?
   -> BGR_131021_NODE_96_length_46113_cov_32.412567: 53
   -> BGR_140121_NODE_54_length_34619_cov_66.823718: 47
   -> BGR_140717_NODE_168_length_31258_cov_37.020094: 44

  What kind of genes does this virus have? Look at the PHROGS_Category. (This can also be solved by using commands)
  BGR_131021_NODE_96_length_46113_cov_32.412567

   DNA, RNA and nucleotide metabolism
   head and packaging
   other
   unknown
 BGR_140121_NODE_54_length_34619_cov_66.823718

   connector
   DNA, RNA and nucleotide metabolism
   head and packaging
   lysis
   other
   tail
   unknown
 BGR_140717_NODE_168_length_31258_cov_37.020094

   connector
   DNA, RNA and nucleotide metabolism
   head and packaging
   lysis
   other
   tail
   unknown


  20. Reset the table so you can see all viruses again. Filter PHROGS_Category to moron, auxiliary metabolic gene and host takeover. If you look at the column PHROGS_GO_Annotation, you can see more general information related to the pathways. What kind of metabolism are the viruses involved in?
   -> Pyrimidine, Purine, sulfur, nicotinade and nicotinamide, Selenocompound, Porphyrin and chlorophyll

  21. While still looking at the results filtered for moron, auxiliary metabolic gene and host takeover: Are there any toxin genes? Briefly look up the function of them (What do they do, where do they occur, what does it mean for this virus and its host?)
   -> Zot-like toxin (Zonular occludens toxin(Zot))
   Function: Zot disrupts tight junctions between epithelial cells, increasing intestinal permeability.

   Where it occurs: Originally described in Vibrio cholerae; also found in some bacteriophages that infect bacteria.

   Implication for virus and host: If a virus carries a Zot-like gene, it may influence the host bacterium’s interaction with its environment (e.g., virulence) or help the virus manipulate host cells. It doesn’t make the virus infect humans directly, but it can affect bacterial pathogenicity.

 **Binning** 
  22. What does each line represent (focus on column 1,2,5)? What is the purpose of this table?
   -> This table lists viral bin members (one virus per line), which are the cluster representatives from module 03. It combines previous results (quality, taxonomy, abundance). For each time point, it shows the covered fraction and RPKM, so we can see how abundant each virus is over time.

  23. How many High-quality "viruses" are there after binning?
   -> 9 -> I just counted in the quality_summary

  24. Thinking for yourself: Filter the results so you can only see vBin_16 (or just highlight the corresponding 5 lines). The metrics for all bin members are the same. But membership_provirus has different values. What does this tell you about the vBin/vMAG 16?
   ->  Binning groups viruses into vMAGs, so each vMAG contains the same virus. Virus 16 appears both as a provirus and free virus, suggesting it can do both lytic and lysogenic cycles.

  25. Thinking for yourself: Are your complete viruses part of a bin? Why does this result make sense? Tip: Use the search function or grep!
   ->  The complete viruses are not part of a bin. The purpose of binning is to restore a fragmented genome by grouping together the individual fragments based on coverage information. Since our complete viruses are, well, complete, we don't need to find their missing pieces. **(umformulieren)**


 **Host prediction**
  26. Understanding the output: Find this table and open it: 08_iPHoP/Host_prediction_to_genome_m90.csv. The input file to generate this table is 08_iPHoP/MVP_07_Filtered_conservative_Prokaryote_Host_Only_best_vBins_Representative_Unbinned_vOTUs_Sequences_iPHoP_Input_clean.fna. Based on the first column of the table and the name of the input file: What does each line represent? (Meaning: What kind of viruses are in this file?)

 Tip: This is a csv file. So you might need to change the delimiter to comma when you open the file.
    -> The input file contains the best vBins (written as vBin_number in the table, with vRhyme_number__sequenceID being individual sequences of that bin), and all unbinned vOTU representatives. Each line is one virus (unbinned vOTU representative, vMAG, or vBin member=binned vOTU representative), where a host could be predicted. Viruses without a prediction are not in the file. Hosts that start with BGR_* are MAGs from the dataset, while all other host identifyers are from reference databases.


  27. What hosts were predicted for the complete viruses? From what habitat did the hosts come from?
    -> (BGR_130829_bin.14.strict,) d__Bacteria;p__Bacillota_A;c__Clostridia;o__Tissierellales;f__Peptoniphilaceae;g__;s__,blast,90.50,iPHoP-RF;78.00
    -> Habitat: Biogasreactor


  28. 

Find an example for a virus with more than one host prediction

    A) With closely related hosts
    B) With distantly related hosts

Be sure to look at the VirusID in column 1 carefully to make sure it is not just different viruses! Tip: You won't have to scroll very far...

[For both A) and B)]: What might be the reason for this result? Discuss based on:

        a. Biological reasoning
        b. Prediction method
    Hint:
       
       Look at the last 3 columns and think about what you learned about host prediction methods earlier today. Explanation of Main method (documentation):

       blast: Score obtained only from blastn hits to host genomes

       iPHoP-RF: Score obtained based on all host-based tools (blastn to host genomes, CRISPR hits, k-mer tools: WIsH, VHM, PHP)

       CRISPR: Score obtained only from CRISPR hits


        c. Potential contamination of the host MAG

Tip: This might be a good addition for your protocol...

A) 
B)
C)







BGR_130305_NODE_1097_length_6048_cov_14.323544 -> der am weitesten entfernte







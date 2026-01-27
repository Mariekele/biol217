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
* `fastq` to get an overview
-> command: `fastqc file.fastq.gz -o output_folder/`
-> `-o` putting the contents of the specific file to the folder, which was created before
* `fastp` 
-> command: `fastp -i sample1_R1.fastq.gz -I sample1_R2.fastq.gz -o outdir/sample1_R1_clean.fastq.gz -O outdir/sample1_R2_clean.fastq.gz -t 6 -q 20 -h sample1.html -R "Sample 1 Fastp Report"`
*



2. Assembly





# [Franco](https://github.com/altsplicer) / [***featureCounts***]


## Overview
This a script used for bam files to obtain gene counts using the featureCounts command in the subread package.
FeatureCounts is a program used to count how many reads mapped to genomic features.
For a tutorial see [link](https://rnnh.github.io/bioinfo-notebook/docs/featureCounts.html)

## Slurm script headers
UCI uses the SLURM scheduler so you must use slurm headers to specify how and where you want the job to run. 
You must also name the script SCRIPT_NAME.sub and then run the script using "Sbatch SCRIPT_NAME.sub" while your working directory is in location of the script being run. 

For a more in depth view on a SLURM job script headers see https://rcic.uci.edu/slurm/examples.html.

Make note that this script uses the free partition but you can use your free 1000 core hours of the lab's core hours by changing the header.
Otherwise your job can be killed in the free partition.
``` bash
#!/bin/bash
#SBATCH --job-name=featureCounts_projectname
#SBATCH -p free
#SBATCH --nodes=1
#SBATCH --mem=36G ## request 100GB of memory
#SBATCH -o /dfs3b/hertel-lab/fcarranz/project_name/counts/featureCounts.out ## the name of the job report output file
#SBATCH -e /dfs3b/hertel-lab/fcarranz/project_name/counts/featureCounts.err ## name of the error file
#SBATCH --mail-user fcarranz@uci.edu
#SBATCH --mail-type=ALL
```

## Script featureCounts
``` bash
#module load the latest subread available in UCI's HPC. You type module load FeatureCounts and tab to complete your typing
#subread contains the featurecounts command
module load subread/2.0.1
#Remember you can always "module avail" to see all the available packages
#module avail
```

## Data directory
``` bash
# The directory where the data (bam files) we want to analyze is located
DATA_DIR=/dfs3b/hertel-lab/fcarranz/project_name/bam_files
``` 
## Output directory
``` bash
# The directory where we want the result files to go
OUT_DIR=/dfs3b/hertel-lab/fcarranz/project_name/counts

#Making the result file directory if it isn't already made
#QC_OUT_DIR refers to the name of the directory being made
mkdir -p ${OUT_DIR}
``` bash

## Execution

Here we are performing a loop that will use each file in our data directory as input, "*" is a wild card symbol and in this context matches any file in the indicated directory.
Each file will be processed with the program "featurecounts", "\" symbol indicates that more options for the program are on the next line.
``` bash
# (--outdir) indicates the output directory for the result files
for FILE in `find ${DATA_DIR} -name \*`; do
    featureCounts \
	-p \
	-f \
	-t exon \
	-g exon_id \
	-O \
	-T 8 \
	-s 2 \
	-J \
	-M \
	-a /data/homezvol2/fcarranz/GTF/v38/Exonfeature.gtf \
	-o ${OUT_DIR} \
	$FILE
done
```
-p Specify that input data contain paired-end reads.

-f Perform read counting at feature level (eg. counting reads for exons rather than genes).

-t  Specify feature type(s) in a GTF annotation, 'exon' by default.

-g Specify attribute type in GTF annotation. 'gene_id' by default.

-O Assign reads to all their overlapping meta-features.

-T Number of the threads. 1 by default.

-s Perform strand-specific read counting.Possible values include: 0 (unstranded), 1 (stranded) and 2 (reversely stranded).

-J Count number of reads supporting each exon-exon junction.

-M  Multi-mapping reads will also be counted.

-a Name and location of an annotation file.

-o Output directory location

$FILE refers to the files to be analyzed

# Please note if any of these commands are not sent in as slurm job but done in the terminal then you need to be in a interactive node, NOT THE LOGIN NODE! You will get in trouble with UCI's HPC staff. 
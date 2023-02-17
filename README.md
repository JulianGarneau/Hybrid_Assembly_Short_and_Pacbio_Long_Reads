# :dragon:Hybrid_Assembly_Short_and_Pacbio_Long_Reads:dragon:
Tutorial to perform an hybrid assembly of bacterial or viral genomes using a combination of short and long reads



### Table of contents (main steps of the procedure)

1.	Converting the pacbio files format to fastq files
2.	Assembling the genome with unicycler


## Workflow overview
 
 
 
 
### STEP 1. CONVERTING THE PACBIO FILE FORMAT TO FASTQ FILES NEEDED BY UNICYCLER
 

If we don't have the fastq file for the Pacbio long reads,you will have to convert the specific PacBio "subreads.bam" files before you can use them here with the Unicycler assembler

To convert Pacbio BAM files to fastq that we will need, we have to install the PacBio tool named bam2fastx (available with conda)

 
 ⚠️ PROGRAMS TO INSTALL 💻 

:small_blue_diamond: bam2fastx

I prefer creating a new conda environment for each tool I install

```
conda create --name bam2fastx

conda activate bam2fastx

conda install -c bioconda bam2fastx

bam2fastq -h
```

<br/>

⚠️ TEMPLATE COMMANDS ⌨️

```
bam2fastq -o AF9_CONVERTED AF9_m64270e_221226_142146.subreads.bam
```




# More info at : https://github.com/PacificBiosciences/bam2fastx


### STEP 2. READ SUBSAMPLING AND FILTERING TO KEEP ONLY THE LARGEST READS SAVE (A LOT) ON COMPUTING TIME


 ⚠️ PROGRAMS TO INSTALL 💻 

:small_blue_diamond: filtlong


```
conda create --name filtlong

conda activate filtlong

conda install -c bioconda filtlong

filtlong --help
```

Long reads are very cool, but if we keep them all, the process will be very heavy and will take many hours/days...

It is recommended to make a first selection of reads to keep only the longest reads, and also the reads with the lowest amount of errors. The idea is to have a good balance between these 2 criteria, and to have sufficient mount of reads to get our good assembly.

We can then implement some general parameters, but we can adjust these later on if we are not satisfied with the assemblies we get.

#### Typical long-reads distribution before and after filtering with "filtlong"


<img width="934" alt="image" src="https://user-images.githubusercontent.com/125351299/219693477-1f8f7fa5-7787-4aa4-8e1d-eafdb8992af0.png">



<br/>

⚠️ TEMPLATE COMMANDS ⌨️


```
filtlong --min_length 1000 --keep_percent 90 --target_bases 175000000 AF9_CONVERTED.fastq.gz | gzip > AF9_FILTERED.fastq.gz
```

#### Quick explanation of the parameters:

--min_length x ← Discard any read which is shorter than x kbp.

--length_weight x ← weight given to the length score (default: 1)

--min_mean_q x  ← minimum mean quality threshold

--keep_percent 90 ← Throw out the worst 10% of reads. This is measured by bp, not by read count. So this option throws out the worst 10% of read bases.

--target_bases x ← Remove the worst reads until only x bp remain, useful for very large read sets. If the input read set is less than x xbp, this setting will have no effect.

input.fastq.gz ← The input long reads to be filtered (must be FASTQ format).

| gzip > output.fastq.gz ← Filtlong outputs the filtered reads to stdout. Pipe to gzip to keep the file size down.


<br/>

If we want to adjust the --target_bases parameter to make it more relevant (and much faster procedure), we can use the following formula:

#### Genome size * expected (or wanted) genome coverage = approximate number of bases needed (target bases)

Example for S. salivarius AF9 : about 2 500 000 bp * 70x coverage (between 50-100x is deemed a goof target) = 175 000 000 total bases (we can now set target_bases to this number)

For more details see : https://github.com/rrwick/Filtlong

Compare the number of reads in the files before filtering and after:

```
zgrep -c '@' AF9_CONVERTED.fastq.gz
zgrep -c '@' AF9_FILTERED.fastq.gz
```
How many reads do you have left? From 730 509 reads to 30 912 reads. Not bad! If those are all long and high-quality reads, we understand that we are gonna save a big amount of computer time now. 

### STEP 3. ASSEMBLING THE GENOME(S) WITH UNICYCLER USING ILLUMINA SHORT READS OR LONG READS OR BOTH (HYBRID ASSEMBLY)



In the case where you have both Illumina short reads and PacBio long reads (converted to fastq.gz)

```
unicycler -1 AF9_R1_001.fastq.gz -2 AF9_R2_001.fastq.gz -l AF9_FILTERED.fastq.gz -o AF9_Hybrid_Assembly --mode normal --threads 10
```

The steps where the long-reads are being aligned is somewhat time consuming. 

Example : in the dataset here, we have 30 912 reads, which required around 25 minutes. If the relationship is linear, without doing the "filtlong" step, we might have needed around 10 hours to complete this step of the procedure.

#### The whole procedure required more or less 2 hours for this AF9 hybrid assembly.

Now we will go in the output folder to check our final assembly and verify if we need to re-run with different more optimal parameters

What can we use to check the quality of the assembly quickly? We can start by counting the number of contigs (fragments) in the final assembly file :

```
zgrep -c '>' AF9_Hybrid_Assembly/assembly.fasta
```



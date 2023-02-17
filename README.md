# :dragon: Hybrid_Assembly_Short_and_Pacbio_Long_Reads :dragon:
Tutorial to perform an hybrid assembly of bacterial or viral genomes using a combination of short and long reads



### Table of contents (main steps of the procedure)

1.	Converting the pacbio files format to fastq files
2.	Assembling the genome with unicycler


## Workflow overview
 
 
 
 
### STEP 1. CONVERTING THE PACBIO FILE FORMAT TO FASTQ FILES
 

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

# More info at : https://github.com/PacificBiosciences/bam2fastx

### STEP 2. ASSEMBLING THE GENOME(S) WITH UNICYCLER USING ILLUMINA SHORT READS OR LONG READS OR BOTH (HYBRID ASSEMBLY)



In the case where you have both Illumina short reads and PacBio long reads (converted to fastq.gz)

```
unicycler -1 AF9_R1_001.fastq.gz -2 AF9_R2_001.fastq.gz -l AF9_CONVERTED.fastq.gz -o AF9_Hybrid_Assembly
```




### Short explanation for this step



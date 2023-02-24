# :dragon:Hybrid_Assembly_Short_and_Pacbio_Long_Reads:dragon:
Tutorial to perform an hybrid assembly of bacterial or viral genomes using a combination of short and long reads



### Table of contents (main steps of the procedure)

1.	Converting the pacbio files format to fastq files
2.	Read subsampling and filtering
3. Assembling the genome(s) with unicycler




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




#### More info at : https://github.com/PacificBiosciences/bam2fastx


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
filtlong --min_length 1000 --keep_percent 90 --target_bases 175000000 AF9_CONVERTED.fastq.gz | gzip > AF9_FILTERED_V1.fastq.gz
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

 ⚠️ PROGRAMS TO INSTALL 💻 

:small_blue_diamond: unicylcer

I prefer creating a new conda environment for each tool I install

```
conda create --name unicycler

conda activate unicycler

conda install -c bioconda unicycler

unicycler -h
```

<br/>

In the case where you have both Illumina short reads and PacBio long reads (converted to fastq.gz)

<br/>

⚠️ TEMPLATE COMMANDS ⌨️

```
unicycler -1 AF9_R1_001.fastq.gz -2 AF9_R2_001.fastq.gz -l AF9_FILTERED_V1.fastq.gz -o AF9_Hybrid_Assembly_V1 --mode normal --threads 10
```

The steps where the long-reads are being aligned is somewhat time consuming. 

Example : in the dataset here, we have 30 912 reads, which required around 25 minutes. If the relationship is linear, without doing the "filtlong" step, we might have needed around 10 hours to complete this step of the procedure.

#### The whole procedure required more or less 2 hours for this AF9 hybrid assembly.

Now we will go in the output folder to check our final assembly and verify if we need to re-run with different more optimal parameters

What can we use to check the quality of the assembly quickly? We can start by counting the number of contigs (fragments) in the final assembly file :

```
zgrep -c '>' AF9_Hybrid_Assembly/assembly.fasta
```


NOTES ON THE 3 DIFFERENT ATTEMPTS:

#### My 1st try yielded:

```
filtlong --min_length 1000 --keep_percent 90 --target_bases 175000000 AF9_CONVERTED.fastq.gz | gzip > AF9_FILTERED_V1.fastq.gz


unicycler -1 AF9_R1_001.fastq.gz -2 AF9_R2_001.fastq.gz -l  AF9_FILTERED_V1.fastq.gz -o AF9_Hybrid_Assembly_V1 --mode normal --threads 10
```

1. length=1849405 (Bacterial genome 1st fragment)
2. length=246127 (Bacterial genome 2nd fragment)
3. length=148826 circular=true (Potentiel large plasmid)
4. length=5029 (The rRNA coding region)

+ 3 very small fragments (lower than 500 bp)



#### My 2nd try with these adjusted parameters gave me a better assembly. The rational behind changing the parameters to these was to change the balance between long reads and high-identity (having fewer high-identity reads, but more long reads, min 7000 bp).

With this aapproach, I got 2 major contigs and a smaller one of 5029 bp (A rRNA-16Sr and RNA-23S ribosomal RNA of course...)

.1 length=2105794 (The bacterial genome)
.2 length=148826, circular=true (Potentiel large plasmid)
.3 length=5029 (The rRNA coding region)

+ 2 very small fragments (lower than 500 bp)


```
filtlong --min_length 7000 --keep_percent 90 --target_bases 200000000 AF9_CONVERTED.fastq.gz | gzip > AF9_FILTERED_V2.fastq.gz


unicycler -1 AF9_R1_001.fastq.gz -2 AF9_R2_001.fastq.gz -l AF9_FILTERED_V2.fastq.gz -o AF9_Hybrid_Assembly_V2 --mode normal --threads 10
```


#### My 3rd try yielded

I will try again one this time by lowering the min_length to 4000 and augmenting the coverage a bit more again.

I also want to try with much less coverage, because sometimes it can also work better (ex: 20x only instead of 70x or 100x)

Not the best to change another parameter here at the same time, but I also want to see if changing the mode to --mode bold might allow the connection of the last hanging contig (the 5029 bp one).

```
filtlong --min_length 4000 --keep_percent 90 --target_bases 300000000 AF9_CONVERTED.fastq.gz | gzip > AF9_FILTERED_V3.fastq.gz


unicycler -1 AF9_R1_001.fastq.gz -2 AF9_R2_001.fastq.gz -l AF9_FILTERED_V3.fastq.gz -o AF9_Hybrid_Assembly_V3 --mode bold --threads 10
```

1. length=2106469 (The bacterial genome)
2. length=148826 circular=true (Potentiel large plasmid)
3. length=5029 (The rRNA coding region)
4. length=776 (tRNA region)

## So far, the 3rd try is the best...

#### My 4th try yielded

I want to try lower coverage this time (20x)

2 500 000 bp * 20x coverage = 50 000 000 total bases (we can now set target_bases to this number)

```
filtlong --min_length 4000 --keep_percent 90 --target_bases 50000000 AF9_CONVERTED.fastq.gz | gzip > AF9_FILTERED_V4.fastq.gz


unicycler -1 AF9_R1_001.fastq.gz -2 AF9_R2_001.fastq.gz -l AF9_FILTERED_V4.fastq.gz -o AF9_Hybrid_Assembly_V4 --mode bold --threads 10
```

Results : 31 contigs....  Really bad.

## Pipeline for the assembly of multiple samples 
This section will be dedicated to the automated procedure for assembling multiple samples concomitantly (using loops)

# LOOP CREATION FOR THE AUTOMATED ASSEMBLY WITH MULTIPLE SAMPLES

```
source ~/anaconda3/etc/profile.d/conda.sh
```

```
for dir in */ ; do

	##### PACBIO PBI FILE CONVERSION PART

	conda activate bam2fastx
    echo "Starting the analysis for" "$dir"
    echo "Moving to the folder" "$dir"
    cd "$dir"
    # Assign the name of the folder to the SAMPLE_NAME (and deleting the / from the name)
    SAMPLE_NAME=$(echo "$dir" | sed 's/\///')
    echo ${SAMPLE_NAME}
    echo "Doing bam2fastq on" "${SAMPLE_NAME}"
    bam2fastq -o ${SAMPLE_NAME}_CONVERTED *.bam
    conda deactivate

    ##### FILTERING THE PACBIO FASTQ FILE

    conda activate filtlong
    SAMPLE_NAME=$(echo "$dir" | sed 's/\///')
    # --target_base has be recalculated depending on the genome being analyzed
    echo "Doing filtlong on" "${SAMPLE_NAME}_CONVERTED"
    filtlong --min_length 4000 --keep_percent 90 --target_bases 300000000 ${SAMPLE_NAME}_CONVERTED.fastq.gz | gzip > ${SAMPLE_NAME}_FILTERED.fastq.gz
    conda deactivate
    zgrep -c '@' ${SAMPLE_NAME}_CONVERTED.fastq.gz > ${SAMPLE_NAME}_CONVERTED_read_counts.txt
    zgrep -c '@' ${SAMPLE_NAME}_FILTERED.fastq.gz > ${SAMPLE_NAME}_FILTERED_counts.txt

    ##### HYBRID ASSEMBLY PART

    conda activate unicycler
    SAMPLE_NAME=$(echo "$dir" | sed 's/\///')
    echo "Doing Unicycler on" "${SAMPLE_NAME}_FILTERED.fastq.gz"
    unicycler -1 ${SAMPLE_NAME}_R1_001.fastq.gz -2 ${SAMPLE_NAME}_R2_001.fastq.gz -l ${SAMPLE_NAME}_FILTERED.fastq.gz -o ${SAMPLE_NAME}_Hybrid_Assembly --mode normal --threads 10
    conda deactivate
    zgrep -c '>' ${SAMPLE_NAME}_Hybrid_Assembly/assembly.fasta

### Coming back to original directory to continue the loop on other folders containing the Pacbio and Illumina reads
    cd ..

done

```



## Installation of conda

wget https://repo.anaconda.com/archive/Anaconda3-2020.07-Linux-x86_64.sh
sh Anaconda3-2020.07-Linux-x86_64.sh
# Follow the prompts on the installer screens.
# If you are unsure about any setting, accept the defaults. You can change them later

## Setup of conda environments

conda create -n fastqc fastqc multiqc
conda create -n trimmomatic trimmomatic
conda create -n novoplasty novoplasty
conda create -n bwa bwa samtools
conda create -n igv igv
conda create -n mdust mdust

## Perform fastQC analysis of the raw sequencing reads

# Activate the fastQC conda environment
conda activate fastqc
# Run fastQC
fastqc *.fastq.gz
# Compile fastQC reports using multiqc
multiqc .

## Trim reads using Trimmomatic

# Activate the Trimmomatic conda environment
conda activate trimmomatic
# Download the Trimmomatic zip file to get the Illumina adapter sequences compatible with Trimmomatic
wget http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.39.zip
# Unzip the Trimmomatic zip file to extract the Illumina adapter sequences compatible with Trimmomatic
unzip Trimmomatic-0.39.zip
# Copy the appropriate Illumina adapter sequences to the current folder
cp Trimmomatic-0.39/adapters/TruSeq3-PE.fa .
# Run Trimmomatic
trimmomatic PE -threads 12 -basein WGS_1.fq -baseout WGS_trimmed_ ILLUMINACLIP:TruSeq3-PE.fa:2:30:10:2:keepBothReads LEADING:3 TRAILING:3 MINLEN:50 > multiQC_out.txt

## Perform fastQC analysis of the trimmed sequencing reads

# Activate the fastQC conda environment
conda activate fastqc
# Run fastQC
fastqc *_1P *_2P
# Compile fastQC reports using multiqc
multiqc -f .

## Assemble the mitochondrial genome using NOVOPlasty

# Activate the NOVOPlasty conda environment
conda activate novoplasty
# Download an example configuration file
wget https://raw.githubusercontent.com/ndierckx/NOVOPlasty/master/config.txt
# Modify the configuration as needed (paths to sequencing reads and mitochondrial seed sequence, etc)
nano config.txt
# Run NOVOPlasty
NOVOPlasty.pl -c config.txt &

## Validate the assembly by mapping the sequencing reads back against the mitochondrial genome

# Activate the BWA conda environment
conda activate bwa
# Index the mitochondrial genome with BWA to enable mapping of reads
bwa index -a is Circularized_assembly_1_Test.fasta
# Run the mem algorithm of BWA to map the sequencing reads back against the mitochondrial genome
bwa mem -t 6 Circularized_assembly_1_Test.fasta WGS_trimmed__1P WGS_trimmed__2P > WGS_trimmed___vs_Circularized_assembly_1_Test.sam
# Convert the SAM format file to BAM format
samtools view -b -S -o WGS_trimmed___vs_Circularized_assembly_1_Test.bam WGS_trimmed___vs_Circularized_assembly_1_Test.sam
# Sort the BAM format file
samtools sort -o WGS_trimmed___vs_Circularized_assembly_1_Test_sorted.bam WGS_trimmed___vs_Circularized_assembly_1_Test.bam
# Index the sorted BAM format file
samtools index WGS_trimmed___vs_Circularized_assembly_1_Test_sorted.bam
# Index the mitochondrial genome FASTA file
samtools faidx Circularized_assembly_1_Test.fasta
# Check the mapping rate and proper pairing rate of the mapping results
samtools flagstat WGS_trimmed___vs_Circularized_assembly_1_Test_sorted.bam
# Activate the IGV conda environment
conda activate igv
# Run IGV to visualise the mapping results.
# Note: Select Circularized_assembly_1_Test.fasta (the mitochondrial genome FASTA file) as the genome
# Note: Load WGS_trimmed___vs_Circularized_assembly_1_Test_sorted.bam (the sorted BAM format file) once the genome is loaded
igv

## Identify repeat-rich subsets using the mDUST program

# Load the mDUST conda environment
conda activate mdust
# Run mDUST on the mitochondrial genome
cat Circularized_assembly_1_Test.fasta | mdust -c > mdust_out.txt


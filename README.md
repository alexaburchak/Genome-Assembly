## <a name="started"></a>Getting Started

```sh
# Set up workflow
mkdir genome_assembly
cd genome_assembly
mkdir Data
mkdir 1_fastqc
mkdir 2_assembly
mkdir 3_quast
mkdir 4_busco

# Download the dataset
cd Data
mv ~/Downloads/SRR13577846.fastq.gz ~/genome_assemblly/Data/SRR13577846.fastq.gz
gunzip SRR13577846.fastq.gz

# Change permissions of Data directory to read only
cd ..
chmod -w Data
ls -l #check permissions 
```
## FastQC
```sh
# Install FastQC v0.12.1
conda install -c bioconda fastqc=0.12.1

# Quality assessment using fastqc
fastqc Data/SRR13577846.fastq -o 1_fastqc 
cd 1_fastq
open SRR13577846_fastqc.html #view fastqc results in default browser (chrome in my case)
```

## Genome assembly with hifiasm
```sh
# Clone hifiasm from github
git clone git@github.com:chhylp123/hifiasm.git
cd hifiasm && make

# Assemble genome using 10 CPUs
# Real time: 955.282 sec; CPU: 8937.662 sec; Peak RSS: 16.676 GB
cd ~/genome_assembly/2_assembly
hifiasm/hifiasm -o SRR13577846 -t 10 Data/SRR13577846.fastq 
```

## Quast quality assessment
```sh
# Install quast with pip 
pip install quast 

# Convert primary contig file to fasta format then assess quality with quast
# Elapsed time: 0:00:01.843107 
cd ~/genome_assembly/2_assembly
awk '/^S/{print ">"$2;print $3}' SRR13577846.bp.p_ctg.gfa > SRR13577846.bp.p_ctg.fa 
quast.py 2_assembly/SRR13577846.bp.p_ctg.fa -o ~/genome_assembly/3_quast/quast_output

# Repeat quast assessment using reference genome from NCBI
# Reference genome can be found here: https://www.ncbi.nlm.nih.gov/datasets/taxonomy/4932/ 
quast.py 2_assembly/SRR13577846.bp.p_ctg.fa -r ncbi_dataset/ncbi_dataset/data/GCA_000146045.2/GCA_000146045.2_R64_genomic.fna -o ~/genome_assembly/3_quast/quast_ref_output -t 4
```

## BUSCO assessment
```sh
# Create new conda environment to install busco 
conda create -n busco -c conda-forge -c bioconda busco=5.6.1
conda activate busco 

# Run busco in genome mode, using a specific lineage dataset (saccharomycetes_odb10) 
busco -i 2_assembly/SRR13577846.bp.p_ctg.fa -m genome -l saccharomycetes_odb10 --cpu 10 -f
```

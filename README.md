# Densovirus_Novel-virus# Create the markdown content for the Densovirus_Novel-virus repository

# 🦠 Densovirus_Novel-virus

This GitHub repository provides a bioinformatics pipeline for virome analysis and genome assembly of a novel densovirus identified in the two-spotted cricket (*Gryllus bimaculatus*).

---

## 📂 Overview

Virome analysis was conducted using individual Illumina paired-end read files, utilizing the **Hecatomb-based (v1.3.2)** pipeline [(Roach et al., 2022)](https://doi.org/10.1038/s41587-022-01190-w) for initial data processing. 

### 🔹 Step 1: Quality Trimming and Host Filtering

- **BBTools (v38.90)** was used to trim reads with:
  - Quality score < Q20
  - Length < 50 bp
- This step also included contaminant and duplicate removal.
- This step also included contaminant and duplicate removal.

```bash
bbduk.sh in1=sample_R1.fastq.gz in2=sample_R2.fastq.gz \
  out1=trimmed_R1.fastq.gz out2=trimmed_R2.fastq.gz \
  qtrim=r trimq=20 minlength=50 ref=adapters.fa ktrim=r k=23 mink=11 hdist=1

minimap2 -ax sr host_genome.fa trimmed_R1.fastq.gz trimmed_R2.fastq.gz | \
  samtools view -b -f 12 -F 256 - > host_filtered.bam


 """### 🔹 Step 2: Clustering and Taxonomic Annotation

Clustered reads were annotated using **MMSeqs2** with the following databases:
- NCBI viral genome database  
- Multi-kingdom nucleotide and protein databases derived from NCBI RefSeq (bacterial, archaeal, viral)

This step allowed the identification of the dominant virus, *Blattodean blattambidensovirus 1*, from the read clusters.

---

#### ⚙️ Command

```bash
mmseqs createdb clustered_reads.fasta clustered_reads_DB
mmseqs search clustered_reads_DB viral_refseq_DB result tmp --threads 8
mmseqs convertalis clustered_reads_DB viral_refseq_DB result result.m8


🔹 Step 3: Viral Assembly
Reads annotated to Blattodean blattambidensovirus 1 (Blattambidensovirus) were assembled using MEGAHIT (v1.2.9).

```bash
megahit -1 virus_reads_R1.fastq.gz -2 virus_reads_R2.fastq.gz -o megahit_output

🔹 Step 4: Reference-based Mapping
The presence of the GbDV genome in each sample was confirmed using Bowtie2 (v2.4.1).

bowtie2-build GbDV_reference.fasta GbDV_index
bowtie2 -x GbDV_index -1 sample_R1.fastq.gz -2 sample_R2.fastq.gz -S mapped.sam


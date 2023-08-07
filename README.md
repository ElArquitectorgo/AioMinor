</p>
AioMinor was implemented in Perl programming language, including a main script for nucletide and amino acid varation frequency in virus genome. It accepts bascalled fastq files derived from Nanopore amplicon sequencing, cleaned/trimmed Illumina amplicon/normal fastq files (single-end or paired-end) with a SARS-CoV-2/other virus genome. By default, AioMinor analyses SARS-CoV-2 based on an NCBI reference genome (NC_045512.2), but the user can also provide customized SARS-CoV-2 or other virus genome as a reference.

## Installation:
**1. Create an environment with one step**
```
git clone https://github.com/Hiscox-lab/AioMinor/AioMinor.git
cd AioMinor
conda env create -f my_environment.yml
source activate AioMinor 
```
**2. Create an environment step by step**

Third party dependencies:
  > samtools(>=1.9)
  > 
  > bowtie2(=2.4.1)
  > 
  > minimap2(=2.24)

All these third party tool dependencies should be exported to PATH, so that AioMinor can find them. 

Perl module dependencies:
```
Getopt::Long
File::Basename
Data::Dumper
IO::File
Math::CDF
List::Util
Text::NSP
Bio::DB::Sam
Bio::SeqIO
```

## Usage

Please see the details of each parameter by:

```
Required options:
  -platform           sequencing platform "nanopore" or "illumina".
  -method             method of sequencing Library Preparation "amplicon" or "cDNA".
  -maxins             maximum fragment length of in your amplicon Library with paired-end sequencing.
  -ref                reference genome sequence in fasta file.
  -codingRegion       coding regions in your reference genome.
  -primerbed          primer scheme in bed file.
  -fq                 fastq(.gz) file for single-end sequencing.
  -fq1                fastq(.gz) file for paired-end sequencing mate 1s.
  -fq2                fastq(.gz) file for paired-end sequencing mate 2s.
  -primerbed          amplicon primer information in bed file.

Optional options:
  -samplename         sample name, "alignment" by default.
  -t/-thread          number of threads, 1 by default.
  -minins             minimum fragment length of in your amplicon Library with paired-end sequencing, "50" by default.
  -o                  output path, "./" by default.
  
  -v/-version         Print version information.
  -h/-help            Print help message.\n\n";
```

## **Examples:**
**1. ARTIC Nanopore sequencing data**

To analyse ARTIC-Nanopore amplicon sequencing data with your chosen primer scheme. Extracts the reads containing the identified leader-TRS junctions in fasta format for pool 1 of amplicon primers (The ARTIC primer_bed can be found in the "primer_bed" folder):

```
perl LeTRS.pl -t 16 -extractfasta -pool 1 -Rtch cDNA -mode nanopore -fa example.fastq.gz -primer_bed path_to_primer_V3.bed -o LeTRS_output
```

Optional:
If "-TRSLindependent" option is added, LeTRS will also identify the TRS Leader independent fusion sites in the reads for pool 1 and pool 2 of amplicon primers (The ARTIC primer_bed can be found in the "primer_bed" folder):

```
perl LeTRS.pl -t 16 -pool 0 -Rtch cDNA -mode nanopore -TRSLindependent -fa example.fastq.gz -primer_bed path_to_primer_V3.bed -o LeTRS_output
```

**2. Direct RNA Nanopore sequencing data**

Extracts the reads containing the identified leader-TRS junctions in fasta format:

```
perl LeTRS.pl -t 16 -extractfasta -Rtch RNA -mode nanopore -fq example.fastq.gz -o LeTRS_output
```

Optional customized leader-TRS junctions analysis:
Use SARS-CoV-2 or other coronavirus genomes as a reference, and extract the reads containing the identified leader-TRS junctions in fasta format (the instruction of making a reference_folder could be found in "readme.txt" of "making_reference_folder_example" folder):

```
perl LeTRS.pl -t 16 -extractfasta -Rtch cDNA -mode nanopore -fq example.fastq.gz -primer_bed path_to_custom_primer.bed -o LeTRS_output -ref reference_folder
```

**3. Illumina sequencing data**

Extracts the read-pairs containing the identified leader-TRS junctions in fasta format for pool 1 and pool 2 of amplicon primers (the ARTIC primer_bed can be found in the "primer_bed" folder):

```
perl LeTRS.pl -t 16 -extractfasta -pool 0 -mode illumina -fq #1.fastq.gz:#2.fastq.gz -primer_bed path_to_primer_V3.bed -o LeTRS_output
```
LeTRS also supports single-end Illumina sequencing data:

```
perl LeTRS.pl -t 16 -extractfasta -pool 0 -mode illumina -fq fastq.gz -primer_bed path_to_primer_V3.bed -o LeTRS_output
```

Optional: To analyse customized bam file reads derived from any platform aligned by using a splicing mapping method.

```
perl LeTRS.pl -t 16 -extractfasta -mode illumina -bam example.bam -o LeTRS_output
```

## **Results**
The results can be found under the "results" folder in output path, with four tables: known_junction.tab, known_junction_details.tab, novel_junction.tab and novel_junction_details.tab.

### **known_junction.tab**

The LeTRS output table for known subgenomic mRNA in the sequencing data. "ref_leader_end" and "peak_leader_end" point to the reference position of the end of the leader and the position of the end of the leader identified in the most common reads (peak count) on the reference genome, and "ref_TRS_start" and "peak_TRS_start" refer to the reference position of the start of the TRS and the position of the start of the TRS identified in the most common reads (peak count) on the reference genome.

### **known_junction_details.tab**

The LeTRS output table for details of known subgenomic mRNA in the sequencing data. "peak_leader" and "peak_TRS_start" point to the leader-TRS junctions in known_junction.tab, "ACGAAC" indicates if there is an ACGAAC sequence in the "TRS_seq" (TRS sequences), "20_leader_seq" refers to the 20 nucleotides before the end of the leader, and "ATG_postion" and "first_orf_aa" refer to the first AUG position and translated orf of the sgmRNA.

### **novel_junction.tab**

The LeTRS output table for novel subgenomic mRNA in the sequencing data. "leader_end" and "TRS_start" refer to the position of the end of the leader and the position of the start of the TRS identified in the reads >10.

### **novel_junction_details.tab**

The LeTRS output table for details of novel subgenomic mRNA in the sequencing data. "peak_leader" and "peak_TRS_start" point to the leader-TRS junctions in novel_junction.tab, "ACGAAC" indicates if there is an ACGAAC sequence in the "TRS_seq" (TRS sequences), "20_leader_seq" refers to the 20 nucleotides before the end of the leader, "AUG_postion" and "first_orf_aa" refer to the first AUG position and translated orf of the sgmRNA, and "known_AUG" indicates if the first AUG position is the same as a known sgmRNA.


### **TRS_L_independent_junction.tab**

If "-TRSLindependent" option is added, LeTRS will also identify the TRS Leader independent fusion sites in the reads.

### **primer_usages.tab**

If the sequencing data were derived from amplicon method, the primers in the reads used for amplification of subgenomic mRNAs will be stored in this file for the amplicon primer pool selected. e.g. nCoV-2019_9_RIGHT:8 means there are 8 reads/read pairs used the primer of "nCoV-2019_9_RIGHT" (a primer name in the ARTIC primer_bed that can be found in the "primer_bed" folder) to amplify the subgenomic mRNA.


## **Plotting**
There is also a perl script that can plot a diagram for the output of LeTRS.pl.

**Examples:**
1. Plotting the value in the column of "peak_count" in "known_junction.tab" or "nb_count" in the "novel_junction.tab. "-count 1" indicates the first number of each row in the column and "-count 2" indicates the second number of each row in the column, and so on.

  ```
  perl LeTRS-plot.pl -count 1 -i known_junction.tab
  ```

2. Plotting the value in the column of "peak_peak_count_ratio" in "known_junction.tab" or "count_ratio" in the "novel_junction.tab. "-count 1" indicates the first number of each row in the column and "-count 2" indicates the second number of each row in the column, and so on.

  ```
  perl LeTRS-plot.pl -ratio 1 -i known_junction.tab
  ```

## Customized leader-TRS junctions and SARS-CoV-2 or other coronavirus genomes as reference sequences.
Please the see the "readme.txt" file in the "making_reference_folder_example" folder.

## Citation 
Xiaofeng Dong, Rebekah Penrice-Randal, Hannah Goldswain, Tessa Prince, Nadine Randle, I'ah Donovan-Banfield, Francisco J. Salguero, Julia Tree, Ecaterina Vamos, Charlotte Nelson, Jordan Clark, Yan Ryan, James P. Stewart, Malcolm G. Semple J. Kenneth Baillie, Peter J. M. Openshaw, Lance Turtle, David A. Matthews, Miles W. Carroll, Alistair C. Darby and Julian A. Hiscox. Analysis of SARS-CoV-2 known and novel subgenomic mRNAs in cell culture, animal model, and clinical samples using LeTRS, a bioinformatic tool to identify unique sequence identifiers. GigaScience 2022 DOI: 10.1093/gigascience/giac045



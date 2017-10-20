# synthetic sgRNA library based functional genomics for prokaryotes: sgRNA library design package

## What is this?
This python script collection is one of the two the software subpackages of CRISPRi functional genomics method for the prokaryotes, used for genome-wide or focused sgRNA library design. The basic description of this program can be found at BioRxiv: https://doi.org/10.1101/129668. Please cite this paper or subsequent peer-reviewed publication if this program is useful to your work.

This script collection is user-friendly for experimental biologists with no or limited programming expertise. Generally, the user only need to download the script and several standard files (genome, annotation, etc), edit a configure file to set several parameters needed for sgRNA design, and type in one command line in a Linux environment (for example, terminal in MacOS) to initiate the design process. The output includes the .fasta file for sgRNA and statistics-describing tables as well as figures to intuitively overview the design result.

Seqmap, developed by Jiang, H., Wong, W.H. (2008) Bioinformatics, 24(20) is used in our program for off-target identification and elimination. We have included an executable file of seqmap in the delivered package. If you want to use the updated version of seqmap, please follow the below instructions:
1. Download the Seqmap source file (for all platform) from http://www-personal.umich.edu/~jianghui/seqmap/ and unzip it.
2. Compile the source code following the README file under the unzipped directory of seqmap.
3. Copy and paste the resulted seqmap executable file to the working directory.

## How to use it?
### Step 1: Installation
1. Install Python version 2.7 or above
2. Install Matplotlib version 2.0.2 or above
3. Install Numpy version 1.13.1 or above

### Step 2: Prepare the necessary files. 
This package provides two options for library design: genome-wide or focused sgRNA library. 
1. **For genome-wide library**
For one microorganisms, three standard files are needed to design the genome-scale sgRNA library for all protein-coding genes (or five if RNA-coding genes are also of interest) by the genome. All these files can be accessed at the FTP site of NCBI: ftp://ftp.ncbi.nlm.nih.gov/genomes/archive/old_refseq/Bacteria/
For example, for *E. coli* K12 MG1655 strain, open the hyperlink above, find the directory for this strain:
ftp://ftp.ncbi.nlm.nih.gov/genomes/archive/old_refseq/Bacteria/Escherichia_coli_K_12_substr__MG1655_uid57779/ 
You can find the content under this directory: [here](./image/NCBI_refseq_E.coli.png)
Download files with .fna (genome as a single contig); .ppt (protein-coding gene annotation) and .ffn (protein-coding gene .fasta file) extensions. In many microorganisms, all genes are encoded by a single chromosome. In these cases, we can only find one file each with abovementioned three extensions, respectively. In other cases where the microorganisms carry several plasmids to encode genes (for example: ftp://ftp.ncbi.nlm.nih.gov/genomes/archive/old_refseq/Bacteria/Methylobacterium_extorquens_AM1_uid57605/ see the content under this directory [here](./image/NCBI_refseq_AM1.png)), you may be interested in designing sgRNA library for all genes. To cope with this, download all files with these extensions and combine those with the same extension, resulting in three combined files with .fna, .ffn and .ptt extension, respectively. Note that .fna and .ptt files have one and three header lines, respectively. In file combination process, remove these header lines and only keep them for the combined file at the very beginning.
If you are also interested in RNA-coding genes, download files with .fna (genome as a single contig); .rnt (RNA-coding gene annotation) and .frn (RNA-coding gene .fasta file) extensions. Follow the same instructions as described above. We strongly recommand to process protein- and RNA-coding gene sgRNA library design seperately (run one process with .fna, .ptt, .ffn and configure files; run another with .fna, .rnt, .frn and configure files;), because different parameters may be needed in the configure file (see below), due to the fact that RNA-coding genes tent to have much smaller coding regions in contrast to their protein-coding counterparts.

2. **For focused library**
Sometimes, only a subset of genes (for example, all transcription factors) are of interest in particular research project. In these cases, you need to construct a [.fasta file](https://en.wikipedia.org/wiki/FASTA_format) for these genes containing all their DNA sequences, similar to .ffn (.frn) file mentioned above. You can name each gene uniquely as you like in the '>' line of this constructed .fasta file. Avoid to use ‘-’, ‘_’ and ‘ ’.(space) in these names. Besides, download .fna file from relevant directory of the studied microorganisms as described above.

3. **Coping with multiple copy issue**
Some genes have multiple copies in the genome. If it is not specified, the program cannot design any sgRNA for such genes, because 'off-target' site due to other copies in the genome can be always identified for any designed sgRNA. Although it is generally a minor issue due to very small number of such genes, we still design a optional method to handle it. If you want to cope with this issue to make your sgRNA library more comprehensive, prepare another file by an all-against-all BLASTN. Install [BLAST+ package](ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/), make nucleotide sequence database for your .ffn (.frn) file by makeblastdb command and then run the following:
"blastn -query your_.ff(r)n_file -db your_.ff(r)n_database -evalue 0.001 -out your_output_name -outfmt 6"
The output file of this command will be used to identify the genes with multiple copies in the genome. The program will regard each cluster of paralogs as functionally identical and design sgRNAs to target all of members of a cluster. For details of the algorithm, see our paper.

### Step 3: Set up the configure file (see example_configure.txt)
The configure file is used to set all the necessary parameters and tell the program where to find necessary files. This file is in a two-column format using tab as delimiter. Each line starts with one word (name of one parameter) separated with the following (setting of this parameter) by a tab delimiter. We describe each parameter as below.

**ORFcutoff**: The sgRNA location within the gene coding region (ORF 5’=0.0, ORF 3’=1.0) (default=0.05, real number belonging to (0.0,1.0)). We find that the sgRNAs exhibit better activities locating within the first 5% of the coding region. Hence, by default, the sgRNA is designed at this region as many as possible.

**sgRNA_number**: The number of sgRNA you want to design for each gene (default=10, positive integer). According to the description of our paper, we find that 5-10 sgRNAs per gene is enough for pooled screening based functional genomics analysis. However, more sgRNAs provide benefit (stronger statistical significance) for genes with only moderate phenotypic effect.

**GCcontentMin**: The minimal GC content of spacer region (percentage) (default=30, positive integer between 0~100 is accepted).

**GCcontentMax**: The maximum GC content of spacer region (percentage) (default=85, positive integer between 0~100 and > GCcontentMin is accepted). 

In previous reports about dCas9 based CRISPRi system, GC content of sgRNA spacer region is found to be correlated with sgRNA activity. Extreme GC content reduces sgRNA activity. Hence, we suggest the abovementioned threshold. In situations of genome with relative low or high GC content, we suggest to adjust the threshold to (10,90). 

**off_threshold**: the off target penalty threshold (default=20), sgRNAs with potential off-target site carrying penalty score lower than the threshold will be eliminated. For the detailed description of the scoring method, please check our paper. Briefly, we suggest off_threshold >= 20 for library design. In situations where more sgRNAs are desired, the threshold can be decreased to 10, where the off-target effect of CRISPRi is still very slight as previously reported (Gilbert Luke et al., Cell 2014).

**strand**: whether the sgRNA is designed targeting (binding) to the template or nontemplate stand of a coding gene (default=nontemplate, nontemplate or template is accepted). It is suggested by previous reports that dCas9 based CRISPRi system used in this work exhibits better activity when targeting to non-template strand in the coding region.

**negative**: choose whether to design negative control sgRNAs (sgRNA with no significant target across the genome, which is used as negative control in the following pooled screening experiment and data analysis) for the experiment(yes or no， default=yes). We strongly recommend to include the negative control sgRNAs. For the description of negative control sgRNA usage, see our paper.

**negative_number**: The number of negative control sgRNA you want to design for the experiment. If negative option is no, select 0 for this option. The default is 400. We recommand min(400, 5% of sgRNA library size) as the number of netagive control sgRNAs. 

**targetFasta**: DNA sequence file for genes of interest (.ffn file of protein-coding genes and .frn file of RNA-coding genes for genome-scale sgRNA library design, or your customized file for focused sgRNA library design, see Step 2)

**indexFile**: the gene sequence annotation file (.ptt file for protein-coding genes and .rnt file for RNA-coding genes, see Step 2).

**genome**: the genome file (.fna file, see Step 2) used for off-target check.

**blastresult**: blastresult

**multiple**:False

**genomewide**:False

**prefix**: prefix used for naming of all output files, keep it simple without any ‘-’, ‘_’ and ‘ ’. For example, ‘design20171001’ is fine.

parameter|value
---------|-----
ORFcutoff|0.05
sgRNA_number|10    
GCcontentMin|30
GCcontentMax|85
off_threshold|20
strand|nontemplate 
negative|yes
negative_number|400
targetFasta|example.ffn
indexFile|example.ptt
genome|example.fna
blastresult|example_blastresult
multiple|yes
genomewide|yes
prefix|example



### Step 4: After editing the configure file as introduced above, cd to the working directory where the scripts and the necessary files (check list: configure file, files for targetFasta, indexFile and genome, all python scripts, seqmap executable file) are located. Type in the command line below:

python sgRNA_desgin_main.py configure.txt

## Output description
All the output files will be located in a directory named after the prefix given in configure.txt, under which the files about negative control sgRNA are under a subfolder named after negative.
The prefix.fasta file contains the the target gene sequences.
The prefix.txt is the .txt format of output.fasta file.
The prefix.N20.fasta contains the sgRNA library sequences (N20).
The prefix.N20.txt is the .txt format of output.N20.fasta file.
The prefix.N20NGG.fasta contains the sgRNA library sequence with PAM in the relevant genome.
The prefix.N20NGG.txt is the .txt format of output.N20NGG.fasta file.
The prefix.sgRNA_statistics.txt file contains the position information within relevant ORF and the GC content of each designed sgRNA.
The prefix.gene_statistics.txt file contains the length and the designed sgRNA numbers of each gene.
Two .png figures use histogram to summarize the basic information of gene (sgRNA number vs. gene) and sgRNA (position vs. sgRNA) statistics.

Under the negative subdirectory:
The prefix_N20_NC_passed.fasta contains the negative control sgRNA sequences.
The prefix_N20_NC_passed.txt is the .txt format of the output_N20_NC_passed.fasta.

We also included example files of Corynebacterium glutamicum ATCC 13032 (RefSeq NC_003450). After unzipping the download, please directly come to Step 4 to run an example process as an initial test (79 C. glutamicum genes, around 30 min running).

A typical genome-wide design process needs around 24h running on a typical laptop (for example, 2.6 GHz processor, 8 GB memory). Please keep the power on until the process ends automatically. 
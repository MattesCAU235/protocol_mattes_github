# Protocol Day8
## Mattes Schultze
### 29.01.2025
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=16
#SBATCH --mem=32G
#SBATCH --time=2:00:00
#SBATCH --job-name=reademption
#SBATCH --output=reademption.out
#SBATCH --error=reademption.err
#SBATCH --partition=base
#SBATCH --reservation=biol217
module load gcc12-env/12.1.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"
cd $WORK
micromamba activate $WORK/.micromamba/envs/08_reademption

#set proxy environment to download the data and use the internet in the backend
#export http_proxy=http://relay:3128
#export https_proxy=http://relay:3128
#export ftp_proxy=http://relay:3128

# create folders
cd $WORK/rnaseq_1
#reademption create --project_path READemption_analysis --species salmonella="Salmonella Typhimurium"

# Download the files
# FTP_SOURCE=ftp://ftp.ncbi.nih.gov/genomes/archive/old_refseq/Bacteria/Salmonella_enterica_serovar_Typhimurium_SL1344_uid86645/
# wget -O READemption_analysis/input/salmonella_reference_sequences/NC_016810.fa $FTP_SOURCE/NC_016810.fna
# wget -O READemption_analysis/input/salmonella_reference_sequences/NC_017718.fa $FTP_SOURCE/NC_017718.fna
# wget -O READemption_analysis/input/salmonella_reference_sequences/NC_017719.fa $FTP_SOURCE/NC_017719.fna
# wget -O READemption_analysis/input/salmonella_reference_sequences/NC_017720.fa $FTP_SOURCE/NC_017720.fna

# #rename the files similar to the genome naming
# sed -i "s/>/>NC_016810.1 /" READemption_analysis/input/salmonella_reference_sequences/NC_016810.fa
# sed -i "s/>/>NC_017718.1 /" READemption_analysis/input/salmonella_reference_sequences/NC_017718.fa
# sed -i "s/>/>NC_017719.1 /" READemption_analysis/input/salmonella_reference_sequences/NC_017719.fa
# sed -i "s/>/>NC_017720.1 /" READemption_analysis/input/salmonella_reference_sequences/NC_017720.fa
# wget -P READemption_analysis/input/salmonella_annotations https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/210/855/GCF_000210855.2_ASM21085v2/GCF_000210855.2_ASM21085v2_genomic.gff.gz

# # unzip the file
# gunzip READemption_analysis/input/salmonella_annotations/GCF_000210855.2_ASM21085v2_genomic.gff.gz
# wget -P READemption_analysis/input/reads http://reademptiondata.imib-zinf.net/InSPI2_R1.fa.bz2
# wget -P READemption_analysis/input/reads http://reademptiondata.imib-zinf.net/InSPI2_R2.fa.bz2
# wget -P READemption_analysis/input/reads http://reademptiondata.imib-zinf.net/LSP_R1.fa.bz2
# wget -P READemption_analysis/input/reads http://reademptiondata.imib-zinf.net/LSP_R2.fa.bz2

#read alignment
reademption align -p 4 --poly_a_clipping --project_path READemption_analysis

# read coverage
reademption coverage -p 4 --project_path READemption_analysis

# gene quantification, no space after comma
reademption gene_quanti -p 4 --features CDS,tRNA,rRNA --project_path READemption_analysis
reademption deseq -l InSPI2_R1.fa.bz2,InSPI2_R2.fa.bz2,LSP_R1.fa.bz2,LSP_R2.fa.bz2 -c InSPI2,InSPI2,LSP,LSP -r 1,2,1,2 --libs_by_species salmonella=InSPI2_R1,InSPI2_R2,LSP_R1,LSP_R2 --project_path READemption_analysis

# visualzation
reademption viz_align --project_path READemption_analysis
reademption viz_gene_quanti --project_path READemption_analysis
reademption viz_deseq --project_path READemption_analysis


micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo
```

IGB download, open resulting files of previous script in IGB, open
.csv file from deseq folder, filter by p-value and look for gene locus in IGB, 

5 examples: gene ahpC, padj, 9.96936374064518e-07, log2 fold change: 4.58587359330422, lfcSE: 0.858456697842906

unknown sym 1552, padj: 9.9647922981604e-07, log2 fold change -2.30560840209674, lfcSE: 0.431540762740954

secG, padj 9.91781026899692e-05, log2 fold change: 2.8349862837487, lfcSE: 0.653025561182066

then we ran another slightly adjusted script for 4 reads of 2 methanosarcina mazei conditions with the following script:

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=32
#SBATCH --mem=64G
#SBATCH --time=0-04:00:00
#SBATCH --job-name=rna_seq_methanosarcina
#SBATCH --output=rna_seq_methanosarcina.out
#SBATCH --error=rna_seq_methanosarcina.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"
cd $WORK
micromamba activate $WORK/.micromamba/envs/08_reademption

#set proxy environment to download the data and use the internet in the backend
#export http_proxy=http://relay:3128
#export https_proxy=http://relay:3128
#export ftp_proxy=http://relay:3128

# create folders
cd $WORK/rnaseq_mm
#reademption create --project_path READemption_analysis --species methanosarcina="methanosarcina mazei"

#download and rename files manually from NCBI, get fasta and gff file from strain (methanosarcina mazei, go1), gff is reference genome, fasta is ref.seq
#read files from paper, find SSR and download fastq files

#read alignment
reademption align --project_path READemption_analysis \
	--processes 32 --segemehl_accuracy 95 \
	--poly_a_clipping \
	--fastq --min_phred_score 25 \
	--progress

# read coverage
reademption coverage --project_path READemption_analysis \
	--processes 32

# gene quantification, no space after comma
reademption gene_quanti --project_path READemption_analysis \
	--processes 32 --features CDS,tRNA,rRNA 
reademption deseq -l mut_R1.fastq.gz,mut_R2.fastq.gz,wt_R1.fastq.gz,wt_R2.fastq.gz -c mut,mut,wt,wt -r 1,2,1,2 --libs_by_species methanosarcina=mut_R1,mut_R2,wt_R1,wt_R2 --project_path READemption_analysis

# visualzation
reademption viz_align --project_path READemption_analysis
reademption viz_gene_quanti --project_path READemption_analysis
reademption viz_deseq --project_path READemption_analysis

micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo
```
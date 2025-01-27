# Protocol Day6
## Mattes Schultze
### 27.01.25

for loop: 

```bash
for i in ./add/absolute/path/*gz; do
    fastq $i -o ./add/absolute/path/output_dir/ -t 16;
    #echo "Process completed"
done
```


script 1: fastqc 

```bash
#!/bin/bash
#SBATCH --job-name=01_fastq
#SBATCH --output=01_fastq.out
#SBATCH --error=01_fastq.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load micromamba/1.4.2
micromamba activate 01_short_reads_qc

# creata new folder for output of qc 
mkdir -p $WORK/genomics/0_raw_reads/short_reads/1_fastqc_raw

for i in $WORK/genomics/0_raw_reads/short_reads/*.gz; do
    fastqc $i -o /work_beegfs/sunam235/genomics/0_raw_reads/short_reads/1_fastqc_raw -t 16; 
done

micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo
```

script 2: fastp

```bash
#!/bin/bash
#SBATCH --job-name=fastp_day6_2
#SBATCH --output=fastp_day6_2.out
#SBATCH --error=fastp_day6_2.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2
micromamba activate 01_short_reads_qc

## 1.1 fastqc raw reads
# mkdir -p $WORK/genomics/1_short_reads_qc/1_fastqc_raw
# for i in *.gz; do fastqc $i -o $WORK/genomics/1_short_reads_qc/1_fastqc_raw -t 16; done

## 1.2 fastp 
mkdir -p /work_beegfs/sunam235/genomics/0_raw_reads/short_reads/2_cleaned_reads
fastp -i /work_beegfs/sunam235/genomics/0_raw_reads/short_reads/241155E_R1.fastq.gz \
 -I /work_beegfs/sunam235/genomics/0_raw_reads/short_reads/241155E_R2.fastq.gz \
 -R /work_beegfs/sunam235/genomics/0_raw_reads/short_reads/2_cleaned_reads/fastp_report \
 -h /work_beegfs/sunam235/genomics/0_raw_reads/short_reads/2_cleaned_reads/report.html \
 -o /work_beegfs/sunam235/genomics/0_raw_reads/short_reads/2_cleaned_reads/241155E_R1_clean.fastq.gz \
 -O /work_beegfs/sunam235/genomics/0_raw_reads/short_reads/2_cleaned_reads/241155E_R2_clean.fastq.gz -t 6 -q 25


micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo
```
script 3 fastqc clean
```bash
#!/bin/bash
#SBATCH --job-name=day6_fastqc_clean_3
#SBATCH --output=day6_fastqc_clean_3.out
#SBATCH --error=0day6_fastqc_clean_3.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2
micromamba activate 01_short_reads_qc

mkdir -p $WORK/genomics/0_raw_reads/short_reads/3_fastqc_cleaned
for i in $WORK/genomics/0_raw_reads/short_reads/2_cleaned_reads/*.gz; do fastqc $i -o $WORK/genomics/0_raw_reads/short_reads/3_fastqc_cleaned -t 16; done
micromamba deactivate
echo "---------short read cleaning completed successfully---------"


micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo
```

How Good is the read quality? after cleaning, phred score of min 25 until 240 bp, problem with per base sequence content

How many reads do you had before trimming and how many do you have now? now 	1613392, before whole	1639549

Did the quality of the reads improve after trimming? yes

then long reads

```bash
#!/bin/bash
#SBATCH --job-name=day6_fastqc_clean_3
#SBATCH --output=day6_fastqc_clean_3.out
#SBATCH --error=0day6_fastqc_clean_3.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"
cd $WORK
micromamba activate .micromamba/envs/02_long_reads_qc

cd $WORK/genomics/0_raw_reads/long_reads/

mkdir $WORK/genomics/0_raw_reads/long_reads/1_fastq_output

NanoPlot --fastq 241155E.fastq.gz -o $WORK/genomics/0_raw_reads/long_reads -t 6 --maxlength 40000 --minlength 1000 --plots kde --format png --N50 --dpi 300 --store --raw --tsv_stats --info_in_report

micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo

#!/bin/bash
#SBATCH --job-name=day6_filt_4
#SBATCH --output=day6_filt_4.out
#SBATCH --error=day6_filt_4.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"
cd $WORK
micromamba activate .micromamba/envs/02_long_reads_qc

mkdir $WORK/genomics/0_raw_reads/long_reads/2_filtlong

filtlong --min_length 1000 --keep_percent 90 $WORK/genomics/0_raw_reads/long_reads/241155E.fastq.gz | gzip > sample1_cleaned_filtlong.fastq.gz
mv sample1_cleaned_filtlong.fastq.gz $WORK/genomics/0_raw_reads/long_reads/2_filtlong

micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo

#!/bin/bash
#SBATCH --job-name=day6_nanoplot_5
#SBATCH --output=day6_nanoplot_5.out
#SBATCH --error=day6_nanoplot_5.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"
cd $WORK
micromamba activate .micromamba/envs/02_long_reads_qc

#Nanoplot cleaned

cd $WORK/genomics/0_raw_reads/long_reads/2_filtlong
mkdir -p $WORK/genomics/0_raw_reads/long_reads/2_filtlong/3_nanoplot_cleaned


NanoPlot --fastq $WORK/genomics/0_raw_reads/long_reads/2_filtlong/*.gz \
 -o $WORK/genomics/2_long_reads_qc/3_nanoplot_cleaned -t 16 \
 --maxlength 40000 --minlength 1000 --plots kde --format png \
 --N50 --dpi 300 --store --raw --tsv_stats --info_in_report
```


How Good is the long reads quality? mean qual 10.4 before, 11.4 after



How many reads do you had before trimming and how many do you have now?
 	15963 before,  	12446 after


```bash
micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo

#!/bin/bash
#SBATCH --job-name=day6_uni_7
#SBATCH --output=day6_uni_7.out
#SBATCH --error=day6_uni_7.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=1:30:00
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2
echo "---------Unicycler Assembly pipeline started---------"
eval "$(micromamba shell hook --shell=bash)"
cd $WORK
micromamba activate .micromamba/envs/03_unicycler

mkdir /work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output


unicycler -1 /work_beegfs/sunam235/genomics/0_raw_reads/short_reads/2_cleaned_reads/241155E_R1_clean.fastq.gz -2 /work_beegfs/sunam235/genomics/0_raw_reads/short_reads/2_cleaned_reads/241155E_R2_clean.fastq.gz -l /work_beegfs/sunam235/genomics/0_raw_reads/long_reads/2_filtlong/sample1_cleaned_filtlong.fastq.gz -o /work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output -t 16

micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo
```

we used fastqc, fastp, nanoplot, fitlong, unicycler

then, quast for assembled fasta file from unicycler

```bash
#!/bin/bash
#SBATCH --job-name=day6_nanoplot_5
#SBATCH --output=day6_nanoplot_5.out
#SBATCH --error=day6_nanoplot_5.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"

cd $WORK
micromamba activate .micromamba/envs/04_checkm_quast

quast.py /work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/assembly.fasta --circos -L --conserved-genes-finding --rna-finding\
     --glimmer --use-all-alignments --report-all-metrics -o /work_beegfs/sunam235/genomics/0_raw_reads/qunicycler_output/quast -t 16





micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo
```
then checkm

```bash
#!/bin/bash
#SBATCH --job-name=day6_checkm_9
#SBATCH --output=day6_checkm_9.out
#SBATCH --error=day6_checkm_9.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"
cd $WORK
micromamba activate .micromamba/envs/04_checkm
checkm data setRoot $WORK/databases/checkm


# Create the output directory if it does not exist
mkdir -p /work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/checkm
cd /work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output

# Run CheckM for this assembly
checkm lineage_wf /work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/ /work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/checkm -x fasta --tab_table --file /work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/checkm/checkm_results -r -t 24
  
# Run CheckM QA for this assembly
checkm tree_qa ./work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/checkm
checkm qa ./work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/checkm/lineage.ms ./work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/checkm/ -o 1 > ./work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/checkm/Final_table_01.csv
checkm qa ./work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/checkm/lineage.ms ./work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/checkm/ -o 2 > ./work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/checkm/final_table_checkm.csv


micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo
```
then checkm2
```bash
#!/bin/bash
#SBATCH --job-name=day6_checkm2_11
#SBATCH --output=day6_checkm2_11.out
#SBATCH --error=day6_checkm2_11.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"
cd $WORK
micromamba activate .micromamba/envs/04_checkm

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"

cd $WORK
micromamba activate .micromamba/envs/04_checkm2

#checkm data setRoot $WORK/databases/checkm2

cd /work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output
mkdir -p /work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/checkm2
checkm2 predict --threads 32 --input /work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/*.fasta --output-directory /work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/checkm2 
micromamba deactivate

echo "---------Assembly Quality Check Completed Successfully---------"

micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo
```

resulting graph: 

![genome_assembly](./resources/Bildschirmfoto%20vom%202025-01-27%2014-23-56.png)

Prokka:

```bash
#!/bin/bash
#SBATCH --job-name=day6_annotate_12
#SBATCH --output=day6_annotate_12.out
#SBATCH --error=day6_annotate_12.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"
cd $WORK
micromamba activate .micromamba/envs/05_prokka


# Run Prokka on the file

cd /work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/

prokka /work_beegfs/sunam235/genomics/0_raw_reads/unicycler_output/assembly.fasta --outdir /work_beegfs/sunam235/genomics/0_raw_reads/4_annotated_genome --kingdom Bacteria --addgenes --cpus 32


micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo
```

gtdb:

```bash
#!/bin/bash
#SBATCH --job-name=day6_gtd_12
#SBATCH --output=day6_gtd_12.out
#SBATCH --error=day6_gtd_12.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"


# 6 Classification-----------------------------------------------------------
echo "---------GTDB Classification Started---------"
# (can not work, maybe due to insufficient memory usage increase the ram in bash script)
cd $WORK
micromamba activate .micromamba/envs/06_gtdbtk
conda env config vars set GTDBTK_DATA_PATH="$WORK/databases/gtdbtk/release220";

cd /work_beegfs/sunam235/genomics/0_raw_reads/4_annotated_genome
# run before mkdir -p /work_beegfs/sunam235/genomics/0_raw_reads/5_gtdb_classification
echo "---------GTDB Classification will run now---------"
gtdbtk classify_wf --cpus 12 --genome_dir /work_beegfs/sunam235/genomics/0_raw_reads/4_annotated_genome --out_dir /work_beegfs/sunam235/genomics/0_raw_reads/5_gtdb_classification --extension .fna --skip_ani_screen
# reduce cpu and increase the ram in bash script in order to have best performance
micromamba deactivate
echo "---------GTDB Classification Completed Successfully---------"

micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo
```

multiqc:

```bash
#!/bin/bash
#SBATCH --job-name=day6_multiqc_14
#SBATCH --output=day6_multiqc_14.out
#SBATCH --error=day6_multiqc_14.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"

cd $WORK
micromamba activate .micromamba/envs/01_short_reads_qc

multiqc -d $WORK/genomics/ -o $WORK/genomics/6_multiqc

micromamba deactivate
jobinfo

# ##----------------- End -------------
module purge
jobinfo
```

final questions:


    How good is the quality of genome? quast: 5 contigs, checkm2: completeness: 99.98, contamination: 0.29, 
    
    checkm: completeness: 98.88	contamination: 0.19

    Why did we use Hybrid assembler? using long reads for closing gaps in short sequence assemblies 

    What is the difference between short and long reads? short reads up to 250 bp, sequenced with high throughput like illumina, long reads: long dna sequences, sequenced with e.g. nanopore sequencing

    Did we use Single or Paired end reads? Why? paired ends for short reads, better quality control, single ends for long sequences, nanopore can only do single end

    Write down about the classification of genome we have used here

    PROKKA_01272025

    Closest genome reference: GCF_004793475.1
	d__Bacteria;p__Bacteroidota;c__Bacteroidia;o__Bacteroidales;f__Bacteroidaceae;g__Bacteroides;s__Bacteroides muris

    closest genome taxonomy:
    d__Bacteria;p__Bacteroidota;c__Bacteroidia;o__Bacteroidales;f__Bacteroidaceae;g__Bacteroides;s__Bacteroides muris


    closest genome reference radius: 95

    closest genome ani: 97.07 (same species, if lower, same genus until 70%, if >99 same strain)

    closest genome af: 0.743


multiqc stuff: 

assembly length: 4.5
 
contigs: 7

CDS: 3883

genes: 2495

average read length: 251, after trimming 245



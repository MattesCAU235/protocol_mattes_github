# Protocol Day 2
## Mattes Schultze
### 21.01.2025

Today we used our first **for in loop**

The first script we used did a quality check on our sequencing data. The script was ran via the caucluster. For that, we logged into the caucluster through the terminal and used 

```bash
sbatch filename.sh
```
to send the file. The follwing bash script was employed for the quality check.

```bash
#!/bin/bash
#SBATCH --job-name=fastqc
#SBATCH --output=fastqc.out
#SBATCH --error=fastqc.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

#load necessary modules
module load gcc12-env/12.1.0
module load micromamba
eval "$(micromamba shell hook --shell=bash)"
export MAMBA_ROOT_PREFIX=$WORK/.micromamba

cd $WORK

micromamba activate .micromamba/envs/00_anvio/

for file in /work_beegfs/sunam235/metagenomics/0_raw_reads/*.gz; do
    fastqc "$file" -o /work_beegfs/sunam235/metagenomics/0_raw_reads/fastqc_output;
done

# ##----------------- End -------------
module purge
jobinfo
```
The next script was used to clean the sequences

```bash
#!/bin/bash
#SBATCH --job-name=fastp
#SBATCH --output=fastp.out
#SBATCH --error=fastp.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

#load necessary modules
module load gcc12-env/12.1.0
module load micromamba
eval "$(micromamba shell hook --shell=bash)"
export MAMBA_ROOT_PREFIX=$WORK/.micromamba

cd $WORK

micromamba activate .micromamba/envs/00_anvio/

fastp -i /work_beegfs/sunam235/metagenomics/0_raw_reads/BGR_130305_mapped_R1.fastq.gz -I /work_beegfs/sunam235/metagenomics/0_raw_reads/BGR_130305_mapped_R2.fastq.gz -R fastp_21_report -o /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/BGR_130305_mapped_R1_clean.fastq.gz -O /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/BGR_130305_mapped_R2_clean.fastq.gz -t 6 -q 20
fastp -i /work_beegfs/sunam235/metagenomics/0_raw_reads/BGR_130708_mapped_R1.fastq.gz -I /work_beegfs/sunam235/metagenomics/0_raw_reads/BGR_130708_mapped_R2.fastq.gz -R fastp_21_report -o /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/BGR_130708_mapped_R1_clean.fastq.gz -O /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/BGR_130708_mapped_R2_clean.fastq.gz -t 6 -q 20
fastp -i /work_beegfs/sunam235/metagenomics/0_raw_reads/BGR_130527_mapped_R1.fastq.gz -I /work_beegfs/sunam235/metagenomics/0_raw_reads/BGR_130527_mapped_R2.fastq.gz -R fastp_21_report -o /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/BGR_130527_mapped_R1_clean.fastq.gz -O /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/BGR_130527_mapped_R2_clean.fastq.gz -t 6 -q 20
# ##----------------- End -------------
module purge
jobinfo
```

Afterwards, we performed a sequence assembly of the cleaned files with the following script.

```bash
#!/bin/bash
#SBATCH --job-name=assembly
#SBATCH --output=assembly.out
#SBATCH --error=assembly.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=25G
#SBATCH --partition=base
#SBATCH --time=5:00:00
#SBATCH --reservation=biol217

#load necessary modules
module load gcc12-env/12.1.0
module load micromamba
eval "$(micromamba shell hook --shell=bash)"
export MAMBA_ROOT_PREFIX=$WORK/.micromamba

cd $WORK

micromamba activate .micromamba/envs/00_anvio/

cd /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output

                                       
megahit -1 BGR_130305_mapped_R1_clean.fastq.gz -1 BGR_130527_mapped_R1_clean.fastq.gz -1 BGR_130708_mapped_R1_clean.fastq.gz -2 BGR_130305_mapped_R2_clean.fastq.gz -2 BGR_130527_mapped_R2_clean.fastq.gz -2 BGR_130708_mapped_R2_clean.fastq.gz \
--min-contig-len 1000 --presets meta-large -m 0.85 -o /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output -t 12        

# ##----------------- End -------------
module purge
jobinfo
```
The script takes a while to run, marking the end of the day.
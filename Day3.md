# Protocol Day3
## Mattes Schultze
### 22.01.2025

Number of total contigs: 55835

![Bandage](./resources/Bandage_graph_contigs.png)

![Bandage](./resources/)

afterwards, the quality was assessed with quast

```bash
#!/bin/bash
#SBATCH --job-name=TASK
#SBATCH --output=TASK.out
#SBATCH --error=TASK.err
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

metaquast -t 6 -o /work_beegfs/sunam235/metagenomics/0_raw_reads/quast_output -m 1000 /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/final.contigs.fa
   

# ##----------------- End -------------
module purge
jobinfo
```

N50 value: 3014
Contigs assembled: 55835
Total length: 142642168

Afterwards, we did a bunch of stuff with the goal of genome binning



first: Mapping, convert the output of the assembly visualization, the final contig file, to a fasta format

```bash
#!/bin/bash
#SBATCH --job-name=anvio
#SBATCH --output=anvio.out
#SBATCH --error=anvio.err
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

anvi-script-reformat-fasta /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/final.contigs.fa -o /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/contigs.anvio.fa --min-len 1000 --simplify-names --report-file name_conversion.txt



# ##----------------- End -------------
module purge
jobinfo
```

then, and index of the contig file is made with bowtie2-build

```bash
bowtie2-build contigs.anvio.fa contigs.anvio.fa.index
```

afterwards, the cleaned reads, the fastp output, is mapped, with bowtie2

```bash
#!/bin/bash
#SBATCH --job-name=bowtie
#SBATCH --output=bowtie.out
#SBATCH --error=bowtie.err
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

cd /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output

bowtie2-build contigs.anvio.fa contigs.anvio.fa.index

cd /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output

bowtie2 --very-fast -x /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/contigs.anvio.fa.index -1 /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/BGR_130305_mapped_R1_clean.fastq.gz -2 /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/BGR_130305_mapped_R2_clean.fastq.gz -S BGR_130305.sam

bowtie2 --very-fast -x /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/contigs.anvio.fa.index -1 /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/BGR_130527_mapped_R1_clean.fastq.gz -2 /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/BGR_130527_mapped_R2_clean.fastq.gz -S BGR_130327.sam

bowtie2 --very-fast -x /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/contigs.anvio.fa.index -1 /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/BGR_130708_mapped_R1_clean.fastq.gz -2 /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/BGR_130708_mapped_R2_clean.fastq.gz -S BGR_130708.sam

#for i in "${*_R1_clean.fastq.gz}"; do
#base=$(basename "$i" _R1_clean.fastq.gz)
#bowtie2 --very-fast -x /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/contigs.anvio.fa.index -1 ${i} -2 ${base}_R2_clean.fastq.gz -S /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/bowtie_output/"${base}".sam 
#done

# ##----------------- End -------------
module purge
jobinfo
```

the result, a map, is then converted from a .sam to a .bam format

```bash
#!/bin/bash
#SBATCH --job-name=samtools
#SBATCH --output=samtools.out
#SBATCH --error=samtools.err
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

for i in *.sam; do
    samtools view -bS $i > "$i".bam;
done


# ##----------------- End -------------
module purge
jobinfo
```

the contig files are then prepared for binning with anvio, resulting in a contig.db file

```bash
#!/bin/bash
#SBATCH --job-name=contig_prep
#SBATCH --output=contig_prep.out
#SBATCH --error=contig_prep.err
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

cd /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output

anvi-gen-contigs-database -f contigs.anvio.fa -o contigs.db -n 'biol217'

# ##----------------- End -------------
module purge
jobinfo
```

in the directory containing this file, a special server can be accessed via the tutorial found in the github folder

there we can see the following graph

![contigs and SCGs](./resources/mapping.png)

afterwards, we did binning with metabat maxbin2

```bash
#!/bin/bash
#SBATCH --job-name=binning_metabat
#SBATCH --output=binning_metabat.out
#SBATCH --error=binning_metabat.err
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


anvi-cluster-contigs -p /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/merged_profiles/PROFILE.db -c /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/contigs.db -C /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/merged_profiles/METABAT2 --driver metabat2 --just-do-it --log-file log-metabat2

anvi-summarize -p /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/merged_profiles/PROFILE.db -c /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/contigs.db -o /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/merged_profiles/SUMMARY_METABAT2 -C METABAT2

# ##----------------- End -------------
module purge
jobinfo
```
```bash
#!/bin/bash
#SBATCH --job-name=maxbin
#SBATCH --output=maxbin.out
#SBATCH --error=maxbin.err
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

anvi-cluster-contigs -p /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/merged_profiles/PROFILE.db -c /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/contigs.db -C MAXBIN2 --driver maxbin2 --just-do-it --log-file log-maxbin2

anvi-summarize -p /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/merged_profiles/PROFILE.db -c /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/contigs.db -o /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/merged_profiles/SUMMARY_MAXBIN2 -C MAXBIN2



# ##----------------- End -------------
module purge
jobinfo
```

quality estimate with anvio

```bash
Number of Archaea according to metabat: 3

Number of Archaea according to maxbin2: 1

METABAT2: archaea percent completing 98.6, redundancy 2.63, matching domain 1

#!/bin/bash
#SBATCH --job-name=anvi_estimate
#SBATCH --output=anvi_estimate.out
#SBATCH --error=anvi_estimate.err
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

anvi-estimate-genome-completeness -c /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/contigs.db -p /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/merged_profiles/PROFILE.db -C METABAT2

anvi-estimate-genome-completeness -p /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/merged_profiles/PROFILE.db -c /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/contigs.db --list-collections

# ##----------------- End -------------
module purge
jobinfo
```
metabat gives better results

archaea high quality 1 with metabat, 3 with maxbin



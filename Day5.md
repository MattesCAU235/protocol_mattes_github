# Protocol Day5
## Mattes Schultze
### 24.01.2025

bash codes used: 

```bash
anvi-run-scg-taxonomy -c /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/contigs.db -T 20 -P 2

anvi-estimate-scg-taxonomy -c /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/contigs.db -p /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/merged_profiles/PROFILE.db --metagenome-mode --compute-scg-coverages --update-profile-db-with-taxonomy > taxonomy.txt

anvi-summarize -p /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/merged_profiles/PROFILE.db -c /work_beegfs/sunam235/metagenomics/0_raw_reads/assembly_output/contigs.db -o /work_beegfs/sunam235/metagenomics/0_raw_reads/fastp_output/merged_profiles/SUMMARY_METABAT2/final_summary -C METABAT2
```

species of archaeum: bin 36: Methanocculleus sp012797575
bin 35: Methanosarcina flavescens
bin 9: Methanoculleus thermohydrogenotrophicum

redundancy still needs revision, otherwise high quality and high completeness


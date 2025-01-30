# Protocol Day9
## Mattes Schultze
### 30.01.2025



MM_RS06595, padj: 1.85258354188567e-31, log2 fold: -1.96373224488551, IfcSE: 0.160749039795906, unidentified protein

MM_RS10290, padj: 3.12545768498605e-21 log2 fold: 1.51150084024606 IfcSE: 0.149879392873678, unidentified protein

MM_RS05720, padj: 9.18511963885975e-13 log2 fold: 1.09291768955949 IfcSE: 0.140233263695676, unidentified protein




### Viromics

question 1: How many viruses are in the BGR_140717 sample

```bash
grep ">" -c $WORK/viromics/01_GENOMAD/BGR_140717/BGR_140717_Proviruses_Genomad_Output/proviruses_summary/proviruses_virus.fna #11

grep ">" -c $WORK/viromics/01_GENOMAD/BGR_140717/BGR_140717_Viruses_Genomad_Output/BGR_140717_modified_summary/BGR_140717_modified_virus.fna #846
```

Proviruses: 11

Viruses: 846

How many Caudoviricetes viruses in the BGR? 


```bash
grep -c "Caudoviricetes" 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv
```
/BGR_130305/.:614
/BGR_130527/.:684
/BGR_130708/.:850
/BGR_130829/.:954
/BGR_130925/.:849
/BGR_131021/.:1079
/BGR_131118/.:609
/BGR_140106/.:368
/BGR_140121/.:575
/BGR_140221/.:717
/BGR_140320/.:468
/BGR_140423/.:358
/BGR_140605/.:527
/BGR_140717/.:559
/BGR_140821/.:337
/BGR_140919/.:476
/BGR_141022/.:401


How many Low-Quality/Medium-quality/High-quality/Complete? focus on BGR_140919

```bash
cd $WORK/viromics/

grep -c "Low-quality" 02_CHECK_V/BGR_140919/MVP_02_BGR_140919_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv

grep -c "Medium-quality" 02_CHECK_V/BGR_140919/MVP_02_BGR_140919_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv

grep -c "High-quality" 02_CHECK_V/BGR_140919/MVP_02_BGR_140919_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv

grep -c "Complete" 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv
```

low: 485

medium: 1

high: 0

complete: only a few, e.g. 140717

length: open in R, sort by compelteness, viral_length = 31258, hallmark_genes = 10

Check how abundant is your complete virus in the different samples (RPKM)? Look into folders 04_READ_MAPPING/Subfolders/BGR_*_CoverM.tsv and find your viruses.

```R
CoverM<-BGR_140717_CoverM

CoverM[CoverM$Contig == "BGR_140717_NODE_168_length_31258_cov_37.020094", ]

```
RPKM = 305.8641 TPM = 1337.773

Now find/filter your complete virus and find the viral hallmark genes with a PHROG annotation (hint: look at the columns).

example annotation: phrog_27577;tail protein;99.9;1E-28;2.6E-33;239.9;0.0;286;3-292;2-306

gene annotation shows function, percent identity, length of the alignment, coordinates and bit scores and E values to determine how good the alignment is


Now look for the category of “moron, auxiliary metabolic gene and host takeover” any toxin genes???? Quickly look up the function of this toxin (hint, vibrio phage and vibrio cholerae host)

moron, auxiliary metabolic gene and host takeover for that sample: not in sample, its actually in BGR_130305_NODE_578_length_8737_cov_7.071758_11

phrog_1041;CTXphi Zot-like toxin;100.0;4.7E-49;1.2E-53;376.4;0.0;348;1-374;1-383

Toxin increases permeability of cells by affecting tight junctions, gives vibrio cholera toxicity 

**How many High-quality viruses after binning in comparison to before binning?

before binning: 5, after binning: 9

vBin_69
vBin_24
vBin_6
vBin_8
vBin_16
vBin_18
vBin_64
vBin_100
vBin_136

**Are any of the identified viral contigs complete circular genomes (based on identifying direct terminal repeat regions on both ends of the genome)?

contig with DTR medium confidence: BGR_140717_NODE_168_length_31258_cov_37.020094, is the only one with DTR in sample 140717

others:

BGR_140121_NODE_54_length_34619_cov_66.823718

BGR_131021_NODE_96_length_46113_cov_32.412567

complete viral genomes host prediction: make complte virome table: in 02 folder

```bash
grep "Complete" */*.tsv > complete.tsv
```

BGR_131118_NODE_583_length_10977_cov_12.647043: host: d__Bacteria;p__Firmicutes_A;c__Clostridia;o__Tissierellales;f__Peptoniphilaceae;g__Anaerosphaera;s__Anaerosphaera multitolerans

host genome id: RS_GCF_004006535.1

BGR_140717_NODE_168_length_31258_cov_37.020094:

host: d__Bacteria;p__Bacillota_A;c__Clostridia;o__Tissierellales;f__Peptoniphilaceae;g__;s__
















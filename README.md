# ONTViSc-hands-on-training
## Example of whole genome sequencing
**Sample MT001, MT002, MT010 and MT011** are samples that were derived using direct cDNA sequencing kit (SQK-DCS109) followed by whole genome sequencing using [Flongle](https://nanoporetech.com/products/sequence/flongle). Double-stranded (ds) cDNA was synthesised using random hexamers.

The samples originate from different plant commodities (citrus, prunus and miscanthus) and contain different virus types:

| Sample name | Host | Virus | Genome type | Dataset |
| --- | --- | ---  | --- | --- |
| MT001 | Citrus | CEVd, (Citrus endogenous pararetrovirus) | sscRNA | /work/hia_mt18005/raw_data/ONT_MinION_NZMPI/MT001_ONT.fastq.gz |
| MT002 | Prunus | PNRSV | ssRNA(+) tripartite | /work/hia_mt18005/raw_data/ONT_MinION_NZMPI/MT002_ONT.fastq.gz |
| MT010 | Miscanthus | MsiMV | ssRNA(+) | /work/hia_mt18005/raw_data/ONT_MinION_NZMPI/MT010_ONT.fastq.gz |
| MT011 | Citrus | CTV, CVd-VI | ssRNA(+), sscRNA| /work/hia_mt18005/raw_data/ONT_MinION_NZMPI/MT011_ONT.fastq.gz |

For these samples, we recommend performing first a direct read homology search using:
- megablast and the NCBI NT database
- direct taxonomic read classification using Kraken2 (nucleotide-based) and Kaiju (protein-based).  
This will provide a quick overview of whether samples are infected.
```
# This command will:
# Check for the presence of adapters
# Filter reads against the reference host
# Perform a direct read homology search using megablast and the NCBI NT database.
# Perform a direct taxonomic read classification using Kraken2 and Kaiju.

nextflow run eresearchqut/ontvisc -resume -profile singularity \
                            --adapter_trimming \
                            --host_filtering \
                            --host_fasta /path/to/host/fasta/file \
                            --analysis_mode read_classification \
                            --kraken2 \
                            --krkdb /path/to/kraken2_db \
                            --kaiju \
                            --kaiju_dbname /path/to/kaiju/kaiju.fmi \
                            --kaiju_nodes /path/to/kaiju/nodes.dmp \
                            --kaiju_names /path/to/kaiju/names.dmp \
                            --megablast --blast_mode ncbi \
                            --blast_threads 8 \
                            --blastn_db /path/to/ncbi_blast_db/nt
```

After checking the results of the direct read approach, check if some viruses are present in high abundance.
You will see that MsiMV in sample MT010 is present in high abundance. For these cases, it is possible to try a de novo assembly or clustering approach to see if we can recover their full genome.
To perform a denovo assembly approach on MT010 with the tool Canu, try the following command:
```
#!/bin/bash -l
#PBS -N ontvisc
#PBS -l select=1:ncpus=2:mem=8gb
#PBS -l walltime=6:00:00

cd $PBS_O_WORKDIR
module load java
NXF_OPTS='-Xms1g -Xmx4g'
nextflow run eresearchqut/ontvisc -resume profile singularity \
                                 --denovo_assembly \
                                 --canu \
                                 --canu_genome_size 0.01m \
                                 --canu_options 'useGrid=false maxInputCoverage=2000 minReadLength=200 minOverlapLength=100' \
                                 --blast_threads 8 \
                                 --blastn_db /path/to/ncbi_blast_db/nt
```
Running this command should enable the recovery of most of the MsiMV genome.
```
>tig00000001 len=9578 reads=523 class=contig suggestRepeat=no suggestBubble=no suggestCircular=no trim=0-9578
AATAAACTCGCAACCTTCGTGATAAAATCACTCCAGAGGCCGTCCGTCTAGTGGCTCGAAGCTAGTAAAA
GAACGTACTAAAGAGTTGGAGGTAAACCTCTCTCCTCTGCAGAAATAATACGTACTTACCACATTATACA
TGTATAACTAACGAGGTACAACCTCACCACAATAGGTAACACATATAATATTACTTTACTGCAAACAAGG
TTTCCTCGCATCAATGGTGCTGTTGCACCCCAAGTAAGGAGTGCATATTACGACTGACATCGCCAGCCGT
GTGACGCTCGGTATTCTCCTGAGCTTCTCCGACATTCCCATCAAGTCCGAACATACGGGTGTTTGAACCA
CGTACTGCTGCTGCTTTCATCTGCATATGGGCTTCCTTTGCACGCACTGGTGTCCTTGACGTCATTTCAT
AGAAATCAAAAGCATAGCGCGCAAGATTAAAGTCGGTTAAGTTTCGCTGAAGACCGTATCTTGGCATGTA
TCGTTCTGTTGCATTGCGATATTCAATATATGCTTCAGCTGCATCACTAAAATGATGCATTATTTGTCTA
AAGGTTGGAGATGCATTCTCTATTATTGGTTTCAAAGGGAAGGTTCTCTGTTCTTCTCCATCCATCATAG
TCCAAACCCCATTAATATTTGGTGAGCAACCGTTTTCTATGCACCAAACCATAAGTCCACTCATGACAAC
TGTCATCTGTGTGTCATTAAGTTCGTATTCCTTTAGAACGGCTGCATACCATCTGTCAAACTCAGCTCTC
GTGGCTCGTGTGTTTGATATGTCTTGTTGTTGTGGCTTGTATCCAAGTAGGAAGTCTAAATGTAAAATAT
TCTTCCCCTTTGCTTGCGGTAAACGCATTTTCTTGGACATTGCTTTAAGTTTAGGAACTGCCACTGTTCC
TGAAGTGCCAGCATCAACATCTTTGTCTTTGTTGCCTTGCTGGGCTTTGCTAGCCTCGGATGCTGCCCTA
CTTGCTGCCTCCTCCTCTTGCTTCTTTTTATTAGCAGCCTCTTGTCTTTGTTTTGCTGCTGCCTCTTCTT
CACTCTTCTTTTTCGCTTCAGCATTCCTCTGCGCTTGAGCTGCGGCTTCTCGTTGTCTCTGTGCCGCTGC
GGCATCCTCACTGGCCTTTCGCTGTGCTTCCGTTTGTTGTCCAGCATCGACAGAACCAGTGGCGGCCTGG
TGGATAACGTCCACTGCCTCATCCGAAACATAATCGGGCAAATCCATAATAAATCGTTTGAAGTATTCCT
CAATTTCGCTTGGTACTACTTCCTGCCCAGTGTAGAGATTTCTTAGTGCTGATTCTGCTATGTATGGCGC
GAGGCCCTCCTGCGCAAGCCCAGAGAATGGTTGCATCTCAAGTATCCAAGCATAGAATTTCCTTATTTCC
TTAAGTAAATCAGGGTAGCCCCACGCTTCAACCATCGACGCACATATCGCCTCTAATCTGTATTGTGGTA
GCACCGATCTATCCCATTCAAGGATTGCAACTATTCTCTCTCTTTCCAGTTTTGGAATATACATACCGTC
TTGTAAGACACCTCTGGTTGACATGAACCACAAATCCGACTTTTCCCTCGTTCTGTTCACAAAGTCAAAA
TTCAAGCCAAGGTTTGAAAACGAAGTTGAAAAATTGTCAAGTAAGTATTCGTAGTCTGGATGCACTGCAA
GAAGCAAATCGTCCCCATTTGCAAACATTTTACAAACATTGTCAATCATGTCCTCGTGGATCCCGTTTGA
TATCATTGAATAGTTAAAAGCAAGTATAACCATGAGCGTGTTATCAACGACTGTTGATGGTTGTCCACTA
TTGTTACCTTTGAATTTCTTAATTACAGATCCATCTGGCGTCGCTATCGGTGTGTACACTATTTCAGTGT
ATAAGTTTTTAAGCATTCTTTCTCCTAACTCCCACGGTTCCATGAATGATAACCTGATGTCCAAAATGGC
GTTTAAGAGATAGGGCGTTAAAGAACTATCGAATTGGGATCCATCGGCGTCGCAATAAATCCAGCCCTCT
GGCAATGCCCTCAATAGACGATCCCATCCTCCATAGAACTTTGTTATTCCAACTGTCCATGGTCCTTCTA
AATGGTGCGAGTAGAATTGATTATTGAAGTCATCAACACACACTTTGCCACCAAGTAATGTTTCCAACGG
GGCGGCCGTGAACGTTCTAGTTTTATTAGCTTCTGTCTTTTCGATTGGTCGAATCTCAGCTTTGAGTGAA
CCATTCCAAATGCCAAGTTTCCCGTAATATAAACGTTCGCACGATTGTTTGATTATTTCAGCCTTATCTT
CGTTTGTAAAATCCTTGAAATAATCTTTCTTTTTACCTGTATATAACGCTCCGACTGCTGCGTTTAAATT
TAGTGACTTGAATATTTCCTCCTCTTCTGTAACATAATTACATTGTTGCATACCAACATTGTGTAAAATA
CGCTTGACTAATTGAACTGCTCTCTTGAAAACATTATCGTCAACTTCACCAATAAACGTTGGCTTCGAGT
ATTTCATGAGATCTTTGGTGAAAGCTGCCTTATTCAGCCTGCTCTTATCATATTTGCCCATGAGAGGTTT
AAAGTACGCGTTTGCTTCTTCGTGTGTTGATAAGTACAAAGAGAAATGTGGACAAGGACCCTTCACAACG
TGTTTAGTCACCAGCTGACCAGGGCATTTCGCGACTACTTGTAGGTTTTGCTCAACATGTTGCGTTAGCC
ACGTATTATGAGTCATGCACTGCTCTGTCACGTCAAAAGTTAAATCTTCAATCAATTTTGCAGTTTTAAA
TAATCCCGTTGGCTCTGAATCAACTAAATTGAGACCATTCCATGAAATTAAATTCGGGTTATAATGCCAT
CCTTTCTCCCATGATTGAGCTCCATTCAACTCACATATATATGATTCGAAATCTGGTGGCACTGCAACGA
AGAAGTTTGTGTTACCGTTCACTGTTGTTAAACTGTGGATACCGACAATGGACTTGTCTCTGACATTTAC
TAATGGTAACCCACATTGCCCTTCAGTCGTAGAGATCCAGTGTTTCCAAAAGGTACTGTTTCCTTTAGGT
GCCGTCACACTACTTTCTGATACAATACAAGAACTGTAATTTTGTTGGAAATTCACTCCAACTAAGCAGA
CTCTATCATTCCTATTTGGCTCTGTGAATTTTAATTTGCGTGGGAAAGGTGGAAAGTCCTTTGGCAGTTG
CATTATTACCATATCCCTCCCAGAGATCGGATGTAGCTTAACCTCTACTGAATTTCTAATTTTGAACAAA
CCTCGTGATGATCTTAAAGTTATTTCACCATTATTGTATCTGAACAAATGCGCTGGCACAATTAAGAATG
ATCCGTATCCGATTGCGAATACACACTTTCGTGTGTCGTTAGAGTGGTTTTCTATCATACATATCTGATT
CGAAATCGGTGTGTAGTCACCTAACCCCAACATCATAGATTTTGCTTCATGTGAAACTCCTTCTTCGTGC
CTGTTGGGCACCTGACTTGCTGGAACAATCTGTGCTTTGCCTGTCTGGCGTAATGTCCCTTCATACTCAG
GGAATCCTGCTATGTTATTGTTTGAAACAACACGTAATGGGACATGTGGAGTTAGGTCTACTCTCAAGGC
ATTCTCTGAACCATTTTGTATAAAGAATGCTCGTATGCCTGGGTTCGCATAGATATGTTGTCGATCTAAT
CTATCATCATCTAGTGCAGCTTCTCTAATTTCGCTGAAATGCTCTTGCACTAGATTAATGTCTGCGTGTA
TTTGTTCATCTAAGGTGGCTCCTGTTAACGGATCGACAAATCGAATAAGATTATAATCTTGTGGATCAAA
TCCATACATCATGTGGAATTTGTGCTGCTTAACTCCGAGTCCTACCTTTGTTCCACCTTTCTTTTCCTTT
TTGATGTATGCTGAGCCAAAGTTCTCTTCTATCGTATCCTTGGAACCCGTGACATCGTAGGCAAATTTAT
TATCGCGTGCCTTTTGGAATTTGAGCTTCTGTCTGCTCCGCTTATTCTTTGCCTGATGCGTAACTTTTGT
CTGCGACCATTTTGCAAACAGAAACCATAACATAGTTAAACCTCCTGTGAAAACACCTGCTGCTATCATT
AAATCTCTTTGAATTAATGGTGCATTCCACCGCCCTTCTAATTCAAGACATTGTGCAGTCGCATCCAAAC
TTTGGTGTATGACAGTGTTTAGAGCACCAATTTCCATTAGATCTTCCGCTTTTGTAAACTGCGCTCCTGT
TCCTTGGAATTCCATCAGCTGGTCTCGAACACGCACAAGTTTTTCGATGTTGCCTTTGGAGTGATCCTTC
ATGTAGCGTGAACTTAACAAGCTGACTATGCCGCTAAGAGAGAAGGCGTGAGATGATGAGGGGTTAGCAG
AGATTGAATTGTAATGATTTCTCTTTGCATGCTCTTCTGCTATTAAGGTATTTATAATCGCAATTGTTCG
TGGCAACGAAAAAGGATCAGTTCGTAGTGTATATGCAACCTTTCCTGCACATGCGCTTGATAAGCGGCCG
TAACAGCTAGTTTGACTATATTGCAAGATAATGTCGTACAATTCTGCGTATAGCTTATCTGGAACACCTC
GTACATAATATGGTATTTTAATATAATCGCTCAATTCCAAATCGCACCCAATCTTATTATAATCACCAAC
TGTCAACCAATTATGTACATTAGTATAAGGTATGGCGTTTGGTCTCAACATGACTACAGAATCCCTAAGT
TTATATCTTTTAAGTTTTTCGTGGATTTGCGGGTGCATTGAACCATCAAACTTAACAAGCTCACACATAA
TAAAAGGTGATAACTCGAATTGCATCATTGTTCGCGCTTGTTTCACTGTGCATTTTGCCAGGTGTGTTGT
TGAAACATTGTGTGTTATCACATTTAGCCCATAAGCAAAACAAAGAAATGCCGCTTCTGTTGCTATCATG
GCGGGTATTTCTTGTAGACCCTTCATCGTCACACCCGAACGGATCACACTGCCGGGTTTTGTTCTTCCAA
CTCGGCCAAGTCGTTGTATACGCTCACCGTACGAGACACTAACCTTTTTGTAAATAACTGCTCTGTTATC
GACGTCTAGTTCTGCTGTAACTTTAAGTCCAAAATCGACAACGACGTCGACATCTAATGTGACACCATTT
TCGATAATATTTGTGGCGACAACAAAACACTTCCTCATTGGAGTTCCATTAGTGTGTATACCATTCGTGT
TTTGCTTCATTGTTCTTCCATCAACTTTAATTACTGAGTAGTGTTGTTCAGTTAATGCTCTTGAGAGTGT
GTCAACCTCGTTGTAGCTAGCAACATACACGAGAATATTGTTTCCATATTTCGTTGCATCTGCTTTTGAT
CCAGTGCCTAGCTCAAGAACGAACTGATGTTGAGTTAGATTCTCACATATATGAATGTCCACCGGATGCT
GAGTTGAAAACTCGCACTCTCTCCCAGGTGGTGTTGCAGAAACTTTAATAACTTTCCCTTTGAATTCGTA
TTTCTTGAGCAAGCAGTAAAAAGCCATCGCTGGAGCTTCCATGATATGGCACTCATCAAATATTATAAAA
TCATAGTTAAAAAGTTTGTCTGGATTATTGGCATACATATGCAGAGCAAATCCTGATGTCATTATTGTAA
TTGGGGTTGAGCCAAATGAACTAAGTCCACGCATCTGCAATGTTGGACTTACATTGAAAGGGGCGCCCTG
TAGTTGCCGACATACATTCTCTGCTAAAGGTCTCGTCGGTTCTAGCAGAAGAACTTTGCCACTCTGGCAT
AAATGGTAGGGTAAGCCCGTTGACTTTCCTGATCCAACAGCTCCTCTCAACAGGAACTCAGTTTCCTGGT
GTGTATGTGCTATATCGACACTTACTGTTGCTGCATTTGCTCTTGTGAATTCAATGAATTTTCCCCGAGA
CGGTAATGTGGTACTGTTCTGTTGTTTTCAAGTTGCTTCGTCCACCATGCTTCGAAAGTTACGTCATTGC
TGAATGTGTCTGCGGGTAGGTCTTGATCCGTATCAAAATCCACGGTTAATTCTTTTTCAGTACTTAAGAT
GAGTGGATCATCGAGTTCTACGCTTTGGTGCTTGACGGGTGTGGTGAAAATTGATGTAAAATCAAGTCTA
GGAAATGTTGAGTTCTGTTCGAATGTGTTAATAACCGTCCTCATTTTATTCAAAACCTTGTACACGGCAT
CACTTTTAACAGGGTCAAAAATCATCGTGATTAATGTGCCCAATGCCATCGCTTGTTCAAGATTTGTCTC
CAGGTGTGATTTGCCTTCGTGGATGACACCTGTTCCCGTAAGCTCTATCGTAGCTTCGATTAATCGTGGG
TGGTTTTCTCTGATATATTCAATAAAATCCTCACTTGTGAGATTTTTATCGTGTACTTTAAGCAACTTTG
CATGGATTGCACGCACTTCGTTGATCTCTAATTCGTATTCATCCTCACGCGCTTGTTTTTGCAATCTTTT
GTAGTCTTGCATTGTAGTTATTATAGTGTTAGCAATTGTGGACAATAAGCTCAAAATAAGAAAGATATGT
ATGAGCCTAAATATGTCAGGAACAAACCAATACATAGTGTTTATAGCTTTAACTCTAAGTTTCTCGCATT
TATCACATAAAACGCGGTGGAGTTTGGTTGAAATAGAGCTGACTCGACTTCGACTTTTCTGCACTAAATT
TGATGCCAGATGCGTAACTGATATATTGTACACAGCGTCTAAATCTACGCTTTTTCTCAGGGTTAAAGAT
GGTTTGTAGTACTTCTTGTGTCTGAACACATACCAAATTTGTTGCAATCGTGCACAATATGTTAAATCTG
CCCATGCCTGATTTAATGTCTCTACGTAACTTTTTTCCATTAGAACATACAGTCTTTCGTCGTACAATGC
ATAGCCTTGCTCAATTAGCTCTTTGTTCATCTCTGACCTTGTTGTCATAGCTTCTAAGTGTGACCATAGC
AGGCGCTTTGCTGGGCTAACATGACTTATTCCTGATACCGCAAAACGCAATTGTGTTGAAGCCTTTTCCA
GTATTTGCATTTGCTGCACTAGCAATTCCGCTTGTGATGTCTCTTTTGCTAAGGCCTCAAGTTGAGCAAA
AATTGCTGCTACTCCTTGATTTTTAACAATCCAGTACGTCATTGCTTGCTCAATGTAATTGTTGTTGTAC
AATGATATAAGGATAGTTGGCGATGCGATCGCCATCATCAACAGAAACGGTTCCTCCTCAATTATCTCTT
TGATCTTCAAAGGTTTAAACATATTCTTTCCAAGAAACTTGAGAAGACTACTAAAAGTTCTATGGGTCAA
AGTTCCGCCTACCACGTAATCGCGCATTTCACTCTCCATCGACTCATACTGCATTTTAATTAATTGACCA
ATTGTGTTTGCTTTCAGGATGTGGAATCCGACACTTAAGGAACCATATGAATCAATTACATGCATCGTTT
TGTGCTCATGATCAACAAGAATTGGTGGTAATTCTGCGTTCTTTATCTCAGGAAACATAACTGATAGTGC
GTAACACGCAGTTGCTACGTCTTTGAGCTTTGGCCATTTTCCAAGTCTCTCAATTAATTCATCACGCAGA
AATTTTGTGTAGTCTTTGGCTGACTCCTCGTTAACGTTAATCATTGCTGCTAAAAATATGTTGATGTAGC
AATATCCATCCTTTGCAATATACATCATTGGTGGATCTGTATTGGGAAGGTCCACAATTTTGGGTCAACT
GAATTGCCTATGGTGATGTGCCCTTTCGTTGGTGGTATGATTTCCGAATAAGCGGGTTGTCCGAATTCCG
TCGTCACACAACAACAAGGGTATACGTATGCATTGTCTACTTTACTCAGACAAGCTTTGTTGAGCGGTTG
TCGTGTAATCGACACTCCTGTGAAGGATTTACGTATACGATCGAAGTCGAGCGGAATGACTAATTTACCA
ATCGACAATTTCCGTTGCCCATTTGGATTGAGTCGTGTTATATGTTTCCCATACGCATCTGACGGGTCAA
CAGCTTCAAAATAATTTGTAAAGAATCGCTTAGCGTGGTACTCACGTTTTCCCCATAAGAAATTACCATT
AACATCTAATTGATTATCACACATCAGTGCTGTGTTTATGGTGCTCTTCGGTGATATTTTATTCCTAAAT
GTTGCTAGAGTGTCTTCTTTCGCTGATTCCTTACGATTCTTGTACCATCGTGAAATCTCTAACAGGGCTG
CGCTTGCTTTCACCGCGTCCTCAACCGTCAAAGTGTTTATCTTCAATAATGCTTCATTTACATCCATTAT
TTGTTTCGATTGATTGTCAGTGTAGTGACCAATCGTTTCAGCAACCTGCCATGAAATCGGTAGATGTTCG
AATTTTGGGTTACACCTCTGTTTTATAAAATGCAACGTTCGTTTCAACTTCTGGTCTTCAGCTAAGTAGT
CACTCTGCTGTTTTTCAACGCGCACTATGTTCTTATACAGTTTATCTCCAAATTCATCATCTGCTAGTTC
AAGATCATCGATGTTACAGTGTTTACATGTAATCTCAAACGTTGAATGAAACAGAATTTCAAGTAACGCC
ATTCGTTTTCCACACTGCTCTAAATTAACAGTTGGTGTATGCTCAGTATGGGTTGTTGATATGTTTCGAT
TCGCAATGTATGCATCAGTGTATCCCTTCCAGAATGCATTAGCCTGATGATCGCTAAAGTGTTCAATCTC
ATTGATGAAATTCACTTCATGTATGCTATTGATTAGCTGTCCATTGGCTTTTCCGCGAATTATAAATAAC
TCACCATTTTGCACGTAGGTTATGCCACTGTATCCTTTACAAATTTTTGGCTCCATCTTACTGGACACTG
TGGAATAGCATTTAGACACTGCCACGATGTTGTCAATCCAGTGAGCACTTAGATTAACATCTTTACGCTT
GAATTGTCCCATTTCATGGCGCGTTCGGCAATGAAGGTATGTTTTCTTATTCCTGGTGAGTAATGAAATT
TTTGTGACGTTACGCCTCTTACTTCCGACTAACTCCACTTTCAACCCTGTGGTCAGGCCTATCTCAAGTG
TGTCTCTGATCAACTTATTGATTGAACTTGTGACACTTTGCTTGGGAGTGTCACTCTTCTGCTTATGTAA
TCGTGCGTATACCAAGTGATCTGTCGGCCTATTGTCTGACTTCTCAACCCATGTACGTCTGAGAGGTGTT
GGTATGGCTATGGGAGTTTCTTCAACCTCCTGCACAAACGTCTTCCTTAAGATTTTTGCATTATGCGCTT
CTGCTCTCTCACGAACATACTCTTGATGTTTCGCGGCAAAATGTTCCATCACCTTCTTAACGTCGCGCGG
ATTGTCCAAGTCTGGCCTCCATTTGTAGGAAACAGTGGTCCACGCTCCTGCCATTACGTCTCCAAAGCTT
CGCTTAGGTTTACAGAGAGTTCTTTCAAGGCACCTTGCGCTTGAACCGTGAACTTCTATCTGAGTTTGAA
CGTGATTGTGCTTAATCGTGTTCGGTTGTGTAGCAATACGTAACTGAACGAAGTACAA
```
This contig shows 99% coverage to OL312763.1.
<p align="left"><img src="images/blast_results.png" width="750"></p>


You can also test teh assembler Flye. You will want to specify the paramater --meta as this sample contains both host and viral sequences which are present at different concentrations.
```
 #!/bin/bash -l
#PBS -N ontvisc
#PBS -l select=1:ncpus=2:mem=8gb
#PBS -l walltime=12:00:00

cd $PBS_O_WORKDIR
module load java
NXF_OPTS='-Xms1g -Xmx4g'
nextflow run eresearchqut/ontvisc -resume profile singularity \
                                  --denovo_assembly \
                                  --flye \
                                  --flye_options '--genome-size 0.01m --meta' \
                                  --blast_threads 8 \
                                  --blastn_db /path/to/ncbi_blast_db/nt
                                                 
```
You can now compare the contigs obtained with Flye and Canu.

## Example of short amplicon product

**Sample ONT009** is a rubus sample which is infected with Rubus yellow net virus. The target is a short amplicon which was derived using degenerate primers that enable to distinguish between endogenous and exogenous RYNVs. The product is ~100 bp and fails Sanger sequencing. 

| Primer Name | Primer type | Primer sequence (5’-3’) |
| --- | --- | --- |
| RdRp-F | Forward | 5’-GGNGARTTYGGNACNTTYTTYTTY-3’ |
| RdRp-R | Reverse | 5’-NCKCCANCCRCARAANARNGG-3’ |


In this scenario, because the amplicon is very short, we recommend using the rattle-based clustering approach as the de novo assembly approach generally fails to recover products which are < 1000 bp. We also recommend using the --adapter_trimming option to make sure no residual adapters are present at the start and end of the sequences.

```
nextflow run eresearchqut/ontvisc -resume profile singularity \
                                  --adapter_trimming \
                                  --analysis_mode clustering \
                                  --rattle_clustering_options '--lower-length 30 --upper-length 120' \
                                  --blast_threads 8 \
                                  --blastn_db /path/to/host/fasta/file
```

## Example of amplicon data derived using 5' and 3' RACE 
For **sample MT483**, 5'and 3' RACE sequencing reactions were derived using a ligation method to amplify overlapping products which cover the full length of a novel genome identified using sRNASeq. The genome size is predicted to be ~7000 bp. Guided by the sequences recovered using sRNASeq, specific primers were used in each RACE which are ~5000 bp products. 
For this example, we are analysing the 5' and 3' RACE together, so you will place all the reads into a single folder. 
You want to run porechop_abi so it detects and removes the 5' and 3' RACE adapters so we select --adapter_trimming. 
Using the --final_primer_check option, a final primer check will be performed after the de novo assembly step to check for the presence of any residual universal RACE primers at the end of the assembled contigs.
You can either blast against NCBI or the predicted nucleotide sequence of the viral genome.

```
nextflow run eresearchqut/ontvisc -resume -profile singularity \
                                 --adapter_trimming \
                                 --analysis_mode denovo_assembly \
                                 --canu \
                                 --canu_options 'useGrid=false' \
                                 --canu_genome_size 0.01m \
                                 --final_primer_check \
                                 --blastn_db /work/hia_mt18005/databases/blastDB/20230606/nt
```

Reads can also be directly mapped to the predicted amplicon sequence if it is known:
```
nextflow run maelyg/ontvisc -resume -profile singularity \
                            --analysis_mode map2ref \
                            --reference /work/eresearch_bio/test_datasets/MT483_amplicon_RACE_new_chemistry_ligation/AobVX.fasta
```

## Example of direct RNA sequencing

The **SRR17660991** sample is from a Nicotiana tabaccum specimen infected with tomato spotted wilt orthotospovirus. This sample is part of a study that was published in 2022 in Frontiers in Microbiology ([https://www.ncbi.nlm.nih.gov/pmc/articles/PMC913109](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9131090/). The raw data (inc fastq files) are available at https://www.ncbi.nlm.nih.gov/sra/?term=SRR17660991. Total RNA was DNAse treated and polyA tailing were applied for further library preparation using Oxford Nanopore Technologies kit DirectRNA (SQK-RNA002) for sequencing.

For this sample, you can use a de novo approach or a clustering approach.


```
nextflow run eresearchqut/ontvisc -resume -profile singularity \
                                  --analysis_mode clustering \
                                  --host_filtering \
                                  --qual_filt --chopper_options '-q 10' \
                                  --blast_threads 8 --blastn_db /work/hia_mt18005/databases/blastDB/20230606/nt \
                                  --rattle_clustering_options '--rna --lower-length 1000 --upper-length 150000' \
                                  --rattle_polishing_options '--rna' \
                                  --host_fasta ~/code/micropipe/test_data/Plant_host_sequences11_ed.fasta
```

## Example of dengue virus sample sequenced at very high depth
This was sequenced using an amplicon approach. The amplicon size is expected to be ~10,000 bp. Raw data is available in folder /work/eresearch_bio/test_datasets/ET300 on Lyra.

The strategy here is to only retain high quality data as it was sequenced at very high depth. After checking the QC profile, we can see that by performing harsh quality filtering step, we should still retain a decent amount of reads. Chopper retains 14,921 reads using the options '-q 18 -l 9500 --maxlength 11000'.
```
nextflow run eresearchqut/ontvisc -resume -profile singularity \
                                  --qual_filt \
                                  --chopper_options '-q 18 -l 9500 --maxlength 11000' \
                                  --blast_threads 8 --blastn_db /work/hia_mt18005/databases/blastDB/20230606/nt \
                                  --clustering  --rattle_clustering_options '--lower-length 9000 --upper-length 11000'
```

# ONTViSc-hands-on-training
## Example of whole genome sequencing
**Sample MT001, MT002, MT010 and MT011** are whole genome samples. 
| Sample name | Host | Host spp | Sequencing library type | Library preparation kit | Virus | Genome type | Dataset |
| --- | --- | --- | --- | --- | --- | --- | --- |
| MT001 | Citrus | Citrus Troyer × Frost-Lisbon | cDNA directly, WGS | direct cDNA sequencing kit (SQK-DCS109), Flongle, double-stranded (ds) cDNA was synthesised using random hexamers | CEVd, (Citrus endogenous pararetrovirus) | sscRNA | /work/hia_mt18005/raw_data/ONT_MinION_NZMPI/MT001_ONT.fastq.gz |
| MT002 | Prunus | Prunus persica | cDNA directly, WGS | direct cDNA sequencing kit (SQK-DCS109), Flongle, double-stranded (ds) cDNA was synthesised using random hexamers | PNRSV | ssRNA(+) tripartite | /work/hia_mt18005/raw_data/ONT_MinION_NZMPI/MT002_ONT.fastq.gz |
| MT011 | Citrus | Citrus medica L. | cDNA directly, WGS | direct cDNA sequencing kit (SQK-DCS109), Flongle, double-stranded (ds) cDNA was synthesised using random hexamers | CTV, CVd-VI | ssRNA(+), sscRNA| /work/hia_mt18005/raw_data/ONT_MinION_NZMPI/MT011_ONT.fastq.gz |

For these we recommend performing a direct read homology search using megablast and the NCBI NT database and direct taxonomic read classification using Kraken2 and Kaiju.
Example:

# Check for presence of adapters
# Filter reads against reference host
# Perform a direct read homology search using megablast and the NCBI NT database.
# Perform a direct taxonomic read classification using Kraken2 and Kaiju.
```
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


## Example of short amplicon product

**Sample ONT009** is a rubus sample which is infected with Rubus yellow net virus. The target is a short amplicon which was derived using degenerate primers that enable to distinguish between endogenous and exogenous RYNVs. The product is ~100 bp and fails Sanger sequencing. 

| Primer Name | Primer type | Primer sequence (5’-3’) |
| --- | --- | --- |
| RdRp-F | Forward | 5’-GGNGARTTYGGNACNTTYTTYTTY-3’ |
| RdRp-R | Reverse | 5’-NCKCCANCCRCARAANARNGG-3’ |


In this scenario, because the amplicon is very short, we recommend using the rattle-based clustering approach as the de novo assembly approach generally fail to recover products which are < 1000 bp. We also recommend using the --adapter_trimming option to make sure no residual adapters are present at the start and end of the sequences.

```
nextflow run researchqut.ontvisc \
            -resume \
            -profile singularity \
            --adapter_trimming \
            --analysis_mode clustering \
            --rattle_clustering_options '--lower-length 30 --upper-length 120' \
            --blast_threads 8 \
            --blastn_db /path/to/host/fasta/file
```

## Example of amplicon data derived using 5' and 3' RACE 
For **sample MT483**, 5'and 3' RACE sequencing reactions were derived using a ligation method to amplify overalpping products which cover the full length of a novel genome identified using sRNASeq. The genome size is predicted to be ~7000 bp. Guided by the sequences recovered using sRNASeq, specific primers were used in each RACE which are ~5000 bp products. 
For this example, we want to run porechop_abi so it detects and removes the 5' and 3' RACE adapters so we select --adapter_trimming. 
Using the --final_primer_check option, a final primer check will be performed after the de novo assembly step to check for the presence of any residual universal RACE primers at the end of the assembled contigs.

```
nextflow run researchqut.ontvisc \
             -resume \
             -profile singularity \
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
nextflow run maelyg/ontvisc -resume -profile {singularity, docker} \
                            --analysis_mode map2ref \
                            --reference /work/eresearch_bio/test_datasets/MT483_amplicon_RACE_new_chemistry_ligation/AobVX.fasta
```

## Example of direct RNA sequencing

The **SRR17660991** sample is from a Nicotiana tabaccum specimen infected with tomato spotted wilt orthotospovirus. This sample is part of a study that was published in 2022 in Frontiers in Microbiology ([*https://www.ncbi.nlm.nih.gov/pmc/articles/PMC913109](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9131090/). The raw data (inc fastq files) are available at https://www.ncbi.nlm.nih.gov/sra/?term=SRR17660991. Total RNA was DNAse treated and polyA tailing were applied for further library preparation using Oxford Nanopore Technologies kit DirectRNA (SQK-RNA002) for sequencing.

For this sample, you can use a de novo approach or a clustering approach.


```
nextflow run maelyg/ontvisc -resume \
                        -profile singularity \
                        --analysis_mode clustering \
                        --host_filtering \
                        --qual_filt --chopper_options '-q 10' \
                        --blast_threads 8 --blastn_db /work/hia_mt18005/databases/blastDB/20230606/nt \
                        --rattle_clustering_options '--rna --lower-length 1000 --upper-length 150000' \
                        --rattle_polishing_options '--rna' \
                        --host_fasta ~/code/micropipe/test_data/Plant_host_sequences11_ed.fasta
```

## Example of dengue virus sample sequenced at very high depth
This was sequenced using an amplicon approach. The amplicon size expected to be ~10,000 bp. Raw data available in folder /work/eresearch_bio/test_datasets/ET300 on Lyra.

The strategy here is to only retain high quality data as it was sequenced at very high depth. After checking the QC profile, we can see that by performing harsh quality filtering step, we should still retain a decent amount of reads. Chopper retains 14,921 reads using the options '-q 18 -l 9500 --maxlength 11000'.
```
nextflow run ~/code/github/main/ontvisc/main.nf -resume \
                                                -profile singularity \
                                                --qual_filt \
                                                --chopper_options '-q 18 -l 9500 --maxlength 11000' \
                                                --blast_threads 8 --blastn_db /work/hia_mt18005/databases/blastDB/20230606/nt \
                                                --clustering  --rattle_clustering_options '--lower-length 9000 --upper-length 11000'
```

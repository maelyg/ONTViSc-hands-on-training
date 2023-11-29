# ONTViSc-hands-on-training
## Example of short amplicon product

**Sample ONT009** is a short amplicon for Rubus yellow net virus (~100 bp) which failed Sanger sequencing. It was derived using degenerate primers.  

| Primer Name | Primer type | Primer sequence (5’-3’) |
| --- | --- | --- |
| RdRp-F | Forward | 5’-GGNGARTTYGGNACNTTYTTYTTY-3’ |
| RdRp-R | Reverse | 5’-NCKCCANCCRCARAANARNGG-3’ |


In this scenario, we recommend using the clustering approach.We recommend using the --adapter_trimming option to make sure no residual adpaters are present at the start and end of the sequences.

```
nextflow run researchqut.ontvisc \
            -resume \
            -profile singularity \
            --adapter_trimming \
            --analysis_mode clustering \
            --rattle_clustering_options '--lower-length 30 --upper-length 120' \
            --blast_threads 8 \
            --blastn_db /work/hia_mt18005/databases/blastDB/20230606/nt
```

## Example of amplicon data derived using 5' and 3' RACE 
**Sample MT483** 5'and 3' RACE sequencing reactions are ~5000 bp amplicon products which were sequenced using a ligation method to amplify a novel genome identified using sRNASeq. The genome size is predicted to be ~7000 bp.  
For this example, we want to run porechop_abi so it detects and removed the 5' and 3' RACE adapters so we select --adapter_trimming. 
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

The **SRR17660991** sample is from a Nicotiana tabaccum specimen infected with tomato spotted wilt orthotospovirus. This sample is part of a study that was published in 2022 in Frontiers in Microbiology ([*https://www.ncbi.nlm.nih.gov/pmc/articles/PMC913109](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9131090/). The raw data (inc fastq files) are available at https://www.ncbi.nlm.nih.gov/sra/?term=SRR17660991. Total RNA was DNAse treated and polyA tailing was applied for further library preparation using Oxford Nanopore Technologies kit DirectRNA (SQK-RNA002) for sequencing

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
This was sequence usin an amplicon approach. The amplicon size expected to be ~10,000 bp. Raw data available in folder /work/eresearch_bio/test_datasets/ET300 on Lyra.

The strategy here is to only retain high quality data. We can be picky here as it was sequenced at very high depth. After checking the QC profile, we can see that by performing harsh quality filtering step, we still retain a decent amount of reads. Chopper retains 14,921 reads using the options '-q 18 -l 9500 --maxlength 11000'.
```
nextflow run ~/code/github/main/ontvisc/main.nf -resume \
                                                -profile singularity \
                                                --qual_filt \
                                                --chopper_options '-q 18 -l 9500 --maxlength 11000' \
                                                --blast_threads 8 --blastn_db /work/hia_mt18005/databases/blastDB/20230606/nt \
                                                --clustering  --rattle_clustering_options '--lower-length 9000 --upper-length 11000'
```

# ONTViSc-hands-on-training
## Example of short amplicon product

**Sample ONT009** is a short amplicon for Rubus yellow net virus (~100 bp) which failed Sanger sequencing. We recommend using the clustering approach
```
nextflow run researchqut.ontvisc \
            -resume \
            --adapter_trimming \
            --clustering \
            --rattle_clustering_options '--lower-length 30 --upper-length 120' \
            --blast_threads 8 \
            --blastn_db /work/hia_mt18005/databases/blastDB/20230606/nt
```

## Example of amplicon data derived using 5' and 3' RACE 
**Sample MT483** 5'and 3' RACE sequencing reactions are ~5000 bp amplicon products which were sequenced using a ligation method to amplify a novel genome identified using sRNASeq. The genome size is predicted to be ~7000 bp For this example, we want to run porechop_abi so it detects and removed the 5' and 3' RACE adapters so we select --adapter_trimming. The --final_primer_check option will perform a last check of race primers after the assembly step.

```
nextflow run researchqut.ontvisc \
             -resume
             -profile singularity
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

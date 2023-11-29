# ONTViSc-hands-on-training
# Example of short amplicon product

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

# Search of metagenomics data for Endornaviruses
 * April 1, 2022
 * Adam Rivers, USDA-ARS-GBRU

## Source data

I used the search data from [Serratus](https://serratus.io/) Edgar, R.C., Taylor, J., Lin, V. et al. Petabase-scale sequence alignment catalyses viral discovery. Nature 602, 142–147 (2022). https://doi.org/10.1038/s41586-021-04332-2

Specifically I used the long RdRp-containing contigs in the files rdrp_contigs.tar.gz

## Search

Grep for Endornaviruses

```{bash}
grep "Endorna" rdrp_contigs.tsv > Enrornavirus_contigs.tsv
```

Cut out the SRA accessions

```{bash}
cut -f 2 Enrornavirus_contigs.tsv  > Endornavirus_SRA_ids.txt
```

Format the ids into a comma separated list and pull out the endornavirus contigs

```{python}
with open("Endornavirus_SRA_ids.txt", 'r') as fin:
    data = []
    for line in fin:
        data.append(line.strip())

from Bio import SeqIO
endo_contigs = []
for seq_record in SeqIO.parse("rdrp_contigs.fa", "fasta"):
    if seq_record.id is in data:
        endo_contigs.append(seq_record)

SeqIO.write(endo_contigs, "endrnavirus_contigs.fa", "fasta")

```

Use the list to write the SQL query in `bigquery_endornavirus_accessions.sql`
and ran it on [Google Big query](https://console.cloud.google.com/bigquery?project=ars-scinet-vsv-salmonella&d=sra&p=nih-sra-datastore&t=metadata&page=table&ws=!1m10!1m4!4m3!1snih-sra-datastore!2ssra!3smetadata!1m4!1m3!1sars-scinet-vsv-salmonella!2sbquxjob_4a4f1901_17fe65377cf!3sUS)


Merge the two datasets in Python pandas:

```{python}
import pandas
contigs = pandas.read_table("Enrornavirus_contigs.tsv", header=None)
col_labels = ["Contig", "SRA", "Length", "Depth", "Category", "NR_label", "NR_pctid", "NR_evalue", "NV_label", "NV_pctid","NV_evalue", "PalmDB_label", "PalmDB_pctid", "phylum", "class", "order", "family", "genus", "species"]
contigs.columns= col_labels
sra = pandas.read_csv("SRA_metadata_enornavirus_accessions.csv")
fulldata = pandas.merge(contigs, sra, how="left", left_on="SRA", right_on="acc")
fulldata.to_csv("Endornavirus_with_metadata.csv")


```

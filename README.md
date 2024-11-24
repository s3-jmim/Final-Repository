******# Final-Repository**************

**Lab 3: Finding Homologs with BLAST**

Clone Repository
bash

```
git clone https://github.com/Bio312/FinalRepository-$MYGIT
```

Create Working Directory

bash

```
mkdir ~/lab03-$MYGIT/Mygene
```
bash

```
cd ~/lab03-$MYGIT/Mygene
```

Download Query Sequence
bash

```
ncbi-acc-download -F fasta -m protein "NP_003381.1"
```

Perform BLAST Search
Typical output:
bash

```
blastp -db ../allprotein.fas -query NP_003381.1.fa -outfmt 0 -max_hsps 1 -out Mygene.blastp.typical.out
```

Tabular output:
bash

```
blastp -db ../allprotein.fas -query NP_003381.1.fa -outfmt "6 sseqid pident length mismatch gapopen evalue bitscore pident stitle" -max_hsps 1 -out Mygene.blastp.detail.out
```

It helps dentify homologous protein sequences in the database of interest NP_003381.1

Filter High-Scoring Matches
bash

```
awk '{if ($6< 1e-30)print $1 }' Mygene.blastp.detail.out > Mygene.blastp.detail.filtered.out
```

This code helps to retain only high-confidence homologs based on e-value which is less than 1e-30

Count Paralogs per Species
bash

```
grep -o -E "^[A-Z]\.[a-z]+" Mygene.blastp.detail.filtered.out | sort | uniq -c
```

The purpose of the lab was to identify the homologous sequences of the starting protein and perform BLAST searches. 
---

**Lab 4: Gene Family Sequence Alignment**

Filter and Extract Homologs
bash

```
seqkit grep --pattern-file ~/lab03-$MYGIT/Mygene/Mygene.blastp.detail.filtered.out ~/lab03-$MYGIT/allprotein.fas | seqkit grep -v -p "carpio" > ~/lab04-$MYGIT/Mygene/Mygene.homologs.fas
```
This bash will help extract and refine the homologous sequences to further the analysis of the paper 

Align Sequences
bash

```
muscle -align ~/lab04-$MYGIT/Mygene/Mygene.homologs.fas -output ~/lab04-$MYGIT/Mygene/Mygene.homologs.al.fas
```

Visualize Alignment
bash

```
Rscript --vanilla ~/lab04-$MYGIT/plotMSA.R ~/lab04-$MYGIT/Mygene/Mygene.homologs.al.fas
```
This code helps to visualize the alignment of NP_003381.1

The purpose of Lab 4 was to align the homologous sequences which will help visualize and identify the conserved regions. 
--

**Lab 5: Gene Family Phylogeny using IQ-TREE**

Prepare Alignment File
bash

```
sed 's/ /_/g' ~/lab04-$MYGIT/Mygene/Mygene.homologs.al.fas | seqkit grep -v -r -p "dupelabel" > ~/lab05-$MYGIT/Mygene/Mygene.homologsf.al.fas
```

Build Phylogenetic Tree
bash

```
iqtree -s ~/lab05-$MYGIT/Mygene/Mygene.homologsf.al.fas -bb 1000 -nt 2
```

Root the Tree
bash
```
gotree reroot midpoint -i ~/lab05-$MYGIT/Mygene/Mygene.homologsf.al.fas.treefile -o ~/lab05-$MYGIT/Mygene/Mygene.homologsf.al.mid.treefile
```

It shows the midpoint root of the unrooted phylogenetic tree

Visualize Tree
bash
```
nw_display -w 1000 -b 'opacity:0' -s ~/lab05-$MYGIT/Mygene/Mygene.homologsf.al.mid.treefile.svg
```
The purpose of Lab 5 is to construct a phylogenetic tree that is based on the alignment constructed in Lab 4 and shows the evolutionary relationships, divergence events, and patterns of gene evolution using maximum likelihood. 

--
**Lab 6: Gene and Species Tree Reconciliation**

Create Python Environment for ETE3
bash
```
mamba create -n my_python27_env python=2.7 conda activate my_python27_env mamba install -y ete3
```

Reconcile Toy Example
bash
```
java -jar ~/tools/Notung-3.0_24-beta/Notung-3.0_24-beta.jar -s toyspecies.tre -g toygenetree.tre --reconcile --speciestag prefix --savepng --events --outputdir ~/lab06-$MYGIT/
```

Generate RecPhyloXML and Visualize
bash
```
python2.7 ~/tools/recPhyloXML/python/NOTUNGtoRecPhyloXML.py -g toygenetree.tre.rec.ntg --include.species -o toygenetree.tre.rec.xml
thirdkind -Iie -f toygenetree.tre.rec.xml -o toygenetree.tre.rec.svg
convert -density 300 toygenetree.tre.rec.svg toygenetree.tre.rec.pdf
```

Reconcile Mygene Family
bash
```
gotree prune -i Mygene.homologsf.outgroupbeta.treefile -o Mygene.homologsf.pruned.treefile H.sapiens_HBG1_hemoglobin_subunit_gamma1 H.sapiens_HBG2_hemoglobin_subunit_gamma2 H.sapiens_HBB_hemoglobin_subunit_beta H.sapiens_HBD_hemoglobin_subunit_delta
java -jar ~/tools/Notung-3.0_24-beta/Notung-3.0_24-beta.jar -s species.tre -g globins.homologsf.pruned.treefile --reconcile --speciestag prefix --savepng --events --outputdir ~/lab06-$MYGIT/globins/
```
The purpose of Lab 6 is to reconcile the gene tree constructed in Lab 5 with a species tree. It helps to identify gene duplication and loss events in NP_003381.1, differentiates orthologs from paralogs, and gives information into how gene evolution aligns with species evolution.
--

**Lab 8: Protein Domain Prediction**

Prepare Unaligned Protein Sequences
bash
```
sed 's/*//' ~/lab04-$MYGIT/Mygene/Mygene.homologs.fas > ~/lab08-$MYGIT/Mygene/Mygene.homologs.fas
```


Run RPS-BLAST
bash
```
rpsblast -query Mygene.homologs.fas -db ~/data/Pfam/Pfam -out Mygene.rps-blast.out -outfmt "6 qseqid qlen qstart qend value stitle" -evalue .0000000001
```

Generate Domain Phylogeny
bash
```
cp ~/lab05-$MYGIT/globins/globins.homologsf.outgroupbeta.treefile ~/lab08-$MYGIT/globins
Rscript --vanilla plotTreeAndDomains.r globins.homologsf.outgroupbeta.treefile globins.rps-blast.out globins.tree.rps.pdf
```

It is important to know how many paralogs are found in each species.
bash
```
grep -o -E "^[A-Z]\.[a-z]+" Mygene.blastp.detail.filtered.out  | sort | uniq -c
```
The purpose of Lab 8 is to identify and annotate protein domains within the homologous sequences of the gene family NP_003381.1. 

# Final-Repository

Lab 3: Finding Homologs with BLAST
Clone Repository
bash
Copy code
git clone https://github.com/Bio312/FinalRepository-$MYGIT
Purpose: Clone the required repository to start the analysis.
Create Working Directory
bash
Copy code
mkdir ~/lab03-$MYGIT/Mygene
cd ~/lab03-$MYGIT/Mygene
Purpose: Organize files for BLAST analysis.
Download Query Sequence
bash
Copy code
ncbi-acc-download -F fasta -m protein "NP_003381.1"
Purpose: Retrieve the sequence for the protein of interest.
Perform BLAST Search
Typical output:
bash
Copy code
blastp -db ../allprotein.fas -query NP_003381.1.fa -outfmt 0 -max_hsps 1 -out Mygene.blastp.typical.out


Tabular output:
bash
Copy code
blastp -db ../allprotein.fas -query NP_003381.1.fa -outfmt "6 sseqid pident length mismatch gapopen evalue bitscore pident stitle" -max_hsps 1 -out Mygene.blastp.detail.out


Purpose: Identify homologous protein sequences in the database.
Filter High-Scoring Matches
bash
Copy code
awk '{if ($6< 1e-30)print $1 }' Mygene.blastp.detail.out > Mygene.blastp.detail.filtered.out
Purpose: Retain only high-confidence homologs based on e-value.
Count Paralogs per Species
bash
Copy code
grep -o -E "^[A-Z]\.[a-z]+" Mygene.blastp.detail.filtered.out | sort | uniq -c
Purpose: Determine the number of paralogs per species.

Lab 4: Gene Family Sequence Alignment
Create Working Directory
bash
Copy code
mkdir ~/lab04-$MYGIT/Mygene
cd ~/lab04-$MYGIT/Mygene
Purpose: Organize files for alignment analysis.
Filter and Extract Homologs
bash
Copy code
seqkit grep --pattern-file ~/lab03-$MYGIT/Mygene/Mygene.blastp.detail.filtered.out ~/lab03-$MYGIT/allprotein.fas | seqkit grep -v -p "carpio" > ~/lab04-$MYGIT/Mygene/Mygene.homologs.fas
Purpose: Extract and refine the homologous sequences.
Align Sequences
bash
Copy code
muscle -align ~/lab04-$MYGIT/Mygene/Mygene.homologs.fas -output ~/lab04-$MYGIT/Mygene/Mygene.homologs.al.fas
Purpose: Generate multiple sequence alignment.
Visualize Alignment
bash
Copy code
Rscript --vanilla ~/lab04-$MYGIT/plotMSA.R ~/lab04-$MYGIT/Mygene/Mygene.homologs.al.fas
Purpose: Create a visual representation of the alignment.

Lab 5: Gene Family Phylogeny using IQ-TREE
Prepare Alignment File
bash
Copy code
sed 's/ /_/g' ~/lab04-$MYGIT/globins/globins.homologs.al.fas | seqkit grep -v -r -p "dupelabel" > ~/lab05-$MYGIT/globins/globins.homologsf.al.fas
Purpose: Remove duplicate labels and prepare the alignment for tree-building.
Build Phylogenetic Tree
bash
Copy code
iqtree -s ~/lab05-$MYGIT/globins/globins.homologsf.al.fas -bb 1000 -nt 2
Purpose: Estimate the maximum likelihood tree with bootstrap support.
Root the Tree
bash
Copy code
gotree reroot midpoint -i ~/lab05-$MYGIT/globins/globins.homologsf.al.fas.treefile -o ~/lab05-$MYGIT/globins/globins.homologsf.al.mid.treefile
Purpose: Midpoint root the unrooted phylogenetic tree.
Visualize Tree
bash
Copy code
nw_display -w 1000 -b 'opacity:0' -s ~/lab05-$MYGIT/globins/globins.homologsf.al.mid.treefile.svg
Purpose: Generate an easily interpretable graphic of the phylogenetic tree.
Lab 6: Gene and Species Tree Reconciliation
Create Python Environment for ETE3
bash
Copy code
mamba create -n my_python27_env python=2.7
conda activate my_python27_env
mamba install -y ete3
Purpose: Set up the environment for ETE3, a phylogenetic tree manipulation package.
Clone Repository
bash
Copy code
git clone https://github.com/Bio312/lab06-$MYGIT
cd lab06-$MYGIT


Purpose: Clone Lab 6 repository to work on gene and species tree reconciliation.
Create Toy Example Files
Species Tree:
bash
Copy code
echo "(frog,(horse,cat)Mammalia)Tetrapoda;" > toyspecies.tre


Gene Tree:
bash
Copy code
echo "((frog_Alpha,(horse_Alpha,cat_Alpha)),(frog_Beta,horse_Beta));" > toygenetree.tre


Purpose: Generate a toy dataset for practice with reconciliation.
View Trees
bash
Copy code
nw_display ~/lab06-$MYGIT/toyspecies.tre
nw_display ~/lab06-$MYGIT/toygenetree.tre


Purpose: Visualize the toy species and gene trees.
Reconcile Toy Example
bash
Copy code
java -jar ~/tools/Notung-3.0_24-beta/Notung-3.0_24-beta.jar -s toyspecies.tre -g toygenetree.tre --reconcile --speciestag prefix --savepng --events --outputdir ~/lab06-$MYGIT/


Purpose: Use Notung to reconcile the toy species and gene trees, estimating duplication and loss events.
Generate RecPhyloXML and Visualize
bash
Copy code
python2.7 ~/tools/recPhyloXML/python/NOTUNGtoRecPhyloXML.py -g toygenetree.tre.rec.ntg --include.species -o toygenetree.tre.rec.xml
thirdkind -Iie -f toygenetree.tre.rec.xml -o toygenetree.tre.rec.svg
convert -density 300 toygenetree.tre.rec.svg toygenetree.tre.rec.pdf


Purpose: Convert the reconciled tree into XML and create a visual representation for interpretation.
Reconcile Globin Gene Family
bash
Copy code
gotree prune -i globins.homologsf.outgroupbeta.treefile -o globins.homologsf.pruned.treefile H.sapiens_HBG1_hemoglobin_subunit_gamma1 H.sapiens_HBG2_hemoglobin_subunit_gamma2 H.sapiens_HBB_hemoglobin_subunit_beta H.sapiens_HBD_hemoglobin_subunit_delta
java -jar ~/tools/Notung-3.0_24-beta/Notung-3.0_24-beta.jar -s species.tre -g globins.homologsf.pruned.treefile --reconcile --speciestag prefix --savepng --events --outputdir ~/lab06-$MYGIT/globins/


Purpose: Prune unwanted groups and reconcile the globin gene tree with the species tree.

Lab 8: Protein Domain Prediction
Set Up Directory
bash
Copy code
mkdir ~/lab08-$MYGIT/globins && cd ~/lab08-$MYGIT/globins


Purpose: Create a directory for Lab 8 and switch to it.
Prepare Unaligned Protein Sequences
bash
Copy code
sed 's/*//' ~/lab04-$MYGIT/globins/globins.homologs.fas > ~/lab08-$MYGIT/globins/globins.homologs.fas


Purpose: Remove stop codons from raw protein sequences.
Run RPS-BLAST
bash
Copy code
rpsblast -query globins.homologs.fas -db ~/data/Pfam/Pfam -out globins.rps-blast.out -outfmt "6 qseqid qlen qstart qend evalue stitle" -evalue .0000000001


Purpose: Identify Pfam domains within protein sequences using RPS-BLAST.
Generate Domain Phylogeny
bash
Copy code
cp ~/lab05-$MYGIT/globins/globins.homologsf.outgroupbeta.treefile ~/lab08-$MYGIT/globins
Rscript --vanilla plotTreeAndDomains.r globins.homologsf.outgroupbeta.treefile globins.rps-blast.out globins.tree.rps.pdf


Purpose: Create a phylogenetic tree annotated with predicted Pfam domains.


It is important to know how may paralogs are found in each species.
```bash
grep -o -E "^[A-Z]\.[a-z]+" Mygene.blastp.detail.filtered.out  | sort | uniq -c
```


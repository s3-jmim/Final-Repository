# Final-Repository

Lab 3: Finding homologs with BLAST KEY

On the command line, clone the lab 3 repository.

```
    git clone https://github.com/Bio312/FinalRepository-$MYGIT
```
You may be asked to enter your GitHub username and password. Git will now clone a copy of today's lab into a folder called FinalRepository-$MYGIT (where \$MYGIT is your GitHub username). Go there:

    cd FinalReporsitory-$MYGIT

The goal for this section of the lab is to find the homologous sequences to the gende "NP_003381.1" 

We are going to create a database for BLAST to use from these sequences. BLAST will  align your query sequence with each of the sequences in the database, and then return high scoring pairs (HSPs).

Create a new working directory
You will be conducting ONE blast searches today: the protein of the interest. Create a folder for the Mygene BLAST search using the `mkdir` command:

```bash
mkdir ~/lab03-$MYGIT/Mygene
```

Now go to this folder:

```bash
cd ~/lab03-$MYGIT/Mygene
```

We will use a  protein from *Homo sapiens* (human) as our query sequence. You can download it  using the following command:
```bash
ncbi-acc-download -F fasta -m protein "NP_003381.1"
```

## Perform the BLAST search 
Now, perform a blast search using the query protein:
```bash
blastp -db ../allprotein.fas -query NP_003381.1.fa -outfmt 0 -max_hsps 1 -out Mygene.blastp.typical.out
```

## Perform a BLAST search, and request tabular output
It createa a more detailed and easier-to-process output of the same analysis.

Type in the following command:
```bash
blastp -db ../allprotein.fas -query NP_003381.1.fa  -outfmt "6 sseqid pident length mismatch gapopen evalue bitscore pident stitle"  -max_hsps 1 -out Mygene.blastp.detail.out
```

This command helps to avoid counting by hand of the total human hits: 
```bash
grep -c H.sapiens Mygene.blastp.detail.out
```

## Filtering the BLAST output for high-scoring putative homologs

Next we need to choose which putative homologs to include. We only want to include high-scoring matches for our analysis to ensure credible homologs.

Let's require the e-value to be less than 1e-30. 

This is the command to filter our output file to satisfy this requirement:

```bash
awk '{if ($6< 1e-30)print $1 }' Mygene.blastp.detail.out > Mygene.blastp.detail.filtered.out
```

### How many paralogs are in each species?

It is important to know how may paralogs are found in each species.
```bash
grep -o -E "^[A-Z]\.[a-z]+" Mygene.blastp.detail.filtered.out  | sort | uniq -c
```


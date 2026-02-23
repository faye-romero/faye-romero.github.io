---
title: "Notebook"
description: "Coding notebook"
---

## Bioinformatics notebook
Code chunks that I've found repeatedly useful when dealing with genomic data.

---

### General

```
# Extract all chromosome/scaffold names from a FASTA file
grep "^>" mygenome.fasta > list_of_scaffolds.txt

# Search a directory + sub-directories for a file name containing a string
# Example: find all files ending in ".bam"
find . -type f -name "*.bam"

# Execute a command for all files in a directory + sub-directories whose name matches a certain pattern
find . -type f -name "pattern" -exec [COMMAND] {} [TARGET] \;
# Example: find all files in current directory ending in ".bam" and move to a directory called target/
find . -type f -name "*.bam" -exec mv {} target/ \;

# Create a symlink
ln -s /path/to/original/file /path/to/target/folder

```
---

### BCFtools

```
#bcftools v1.21

# Count how many variants in a VCF
bcftools view -H file.vcf.gz | wc -l
#-H = no header

# Get a list of variants in a VCF (CHROM, POS, ID)
bcftools query -f '%CHROM\t%POS\t%ID\n' file.vcf.gz

# Get list of individuals in a VCF
bcftools query -l file.vcf.gz

# Pull all information for one variant
bcftools view -H -i 'ID=="SNPid"' file.vcf.gz

# Pull all information for one sample
bcftools view -s SampleID file.vcf.gz

# Remove or keep a list of variants from a VCF (list is newline-separated)
bcftools view -e 'ID == @remove.txt' file.vcf.gz -Oz -o out.vcf.gz
bcftools view -i 'ID == @keep.txt' file.vcf.gz -Oz -o out.vcf.gz

# Pull only CHROM, POS, ID, REF, ALT, and GT
bcftools query -f '%CHROM\t%POS\t%ID\t%REF\t%ALT[\t%GT]\n' file.vcf.gz > genotype_table.txt

# Create a general site summary table (will be very large!)
echo -e "CHROM\tPOS\tID\tREF\tALT\tGT:AD:DP:GQ" > summary_table.txt
bcftools query -f '%CHROM\t%POS\t%ID\t%REF\t%ALT[\t%GT:%AD:%DP:%GQ]\n' file.vcf.gz >> summary_table.txt
```

---

### Samtools

```
#samtools v1.21

# SAM to BAM
samtools view -S -b samfile.sam > bamfile.bam
# BAM to SAM
samtools view -h -o samfile.sam bamfile.bam

# Get mean read depth from an alignment
samtools depth -a file.bam | awk '{c++;s+=$3}END{print s/c}'

# Extract alignments that only mapped to a specific chromosome
samtools view -b file.bam "Chr1" > Chr1_alns.bam

# Extract a specific region from a FASTA
samtools faidx file.fasta #index first
samtools faidx file.fasta Chr1:1-1000 > Chr1_1-1000.fasta

# Extract chromosome/scaffold names and lengths, sort by length in descending order
samtools faidx file.fasta #index first
cut -f1-2 file.fai | sort -n -r -k 2 > scaffold_lengths.txt

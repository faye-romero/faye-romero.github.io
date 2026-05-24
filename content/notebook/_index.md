---
title: "Notebook"
description: "Coding notebook"
---

## Bioinformatics notebook
Code chunks that I've found repeatedly useful when dealing with genomic data.

---

### BCFtools

```
#bcftools v1.21

# file type conversion
bcftools view -Ou -o file.bcf file.vcf.gz #compressed VCF to uncompressed BCF
bcftools view -Oz -o file.vcf.gz file.bcf #BCF to compressed VCF

# check if a BCF is uncompressed or not (because both file types just end in .bcf)
xxd file.bcf | head -2
# If first line starts with "0000000: 1f8b", the file is BGZF-compressed

# Count how many sites in a VCF
bcftools view -H file.vcf.gz | wc -l
bcftools index --nrecords file.vcf.gz #if VCF is indexed; much faster

# Count how many chromosomes/scaffolds in a VCF
bcftools query -f '%CHROM\n' file.vcf.gz | uniq | wc -l
bcftools index --stats file.vcf.gz | cut -f1 | wc -l #if VCF is indexed; much faster

# Get a list of variants in a VCF (CHROM, POS, ID)
bcftools query -f '%CHROM\t%POS\t%ID\n' file.vcf.gz

# Get list of individuals in a VCF
bcftools query -l file.vcf.gz 

# Subset a VCF to only include a list of individuals (list is newline-separated)
bcftools view -S list.txt file.vcf.gz
# to exclude, use -S ^list.txt

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

# View allele count (AC), allele number (AN), and allele frequency (AF) for a variant; AF = AC/AN
bcftools view file.vcf.gz | bcftools query -i 'ID="snp1"' -f '%ID\tAC=%INFO/AC\tAN=%INFO/AN\tAF=%INFO/AF\n'
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
```

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

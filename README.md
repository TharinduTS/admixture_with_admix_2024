# admixture_with_admix_2024
This is based on 

https://speciationgenomics.github.io/ADMIXTURE/

I am working in beluga and started by copying vcf files there

Then I combined them
```bash
#!/bin/sh
#SBATCH --job-name=fst
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=2:00:00
#SBATCH --mem=30gb
#SBATCH --output=abba.%J.out
#SBATCH --error=abba.%J.err
#SBATCH --account=def-ben

#SBATCH --mail-user=premacht@mcmaster.ca
#SBATCH --mail-type=BEGIN
#SBATCH --mail-type=END
#SBATCH --mail-type=FAIL
#SBATCH --mail-type=REQUEUE
#SBATCH --mail-type=ALL

module load vcftools

vcf-concat combined_Chr10.g.vcf.gz_Chr10_GenotypedSNPs.vcf.gz_filtered.vcf.gz_filtered_removed.vcf.gz combined_Chr1.g.vcf.gz_Chr1_GenotypedSNPs.vcf.gz_filtered.vcf.gz_filtered_removed.vcf.gz combined_Chr2.g.vcf.gz_Chr2_GenotypedSNPs.vcf.gz_filtered.vcf.gz_filtered_removed.vcf.gz combined_Chr3.g.vcf.gz_Chr3_GenotypedSNPs.vcf.gz_filtered.vcf.gz_filtered_removed.vcf.gz combined_Chr4.g.vcf.gz_Chr4_GenotypedSNPs.vcf.gz_filtered.vcf.gz_filtered_removed.vcf.gz combined_Chr5.g.vcf.gz_Chr5_GenotypedSNPs.vcf.gz_filtered.vcf.gz_filtered_removed.vcf.gz combined_Chr6.g.vcf.gz_Chr6_GenotypedSNPs.vcf.gz_filtered.vcf.gz_filtered_removed.vcf.gz combined_Chr7.g.vcf.gz_Chr7_GenotypedSNPs.vcf.gz_filtered.vcf.gz_filtered_removed.vcf.gz combined_Chr8.g.vcf.gz_Chr8_GenotypedSNPs.vcf.gz_filtered.vcf.gz_filtered_removed.vcf.gz combined_Chr9.g.vcf.gz_Chr9_GenotypedSNPs.vcf.gz_filtered.vcf.gz_filtered_removed.vcf.gz | bgzip -c > trop_WGS_all_20_samples_all_chrs.vcf.gz
```

removed not needed samples
```bash
vcftools --remove-indv F_Nigeria_EUA0331_combined__sorted.bam --remove-indv F_Nigeria_EUA0333_combined__sorted.bam --remove-indv M_Nigeria_EUA0334_combined__sorted.bam --remove-indv M_Nigeria_EUA0335_combined__sorted.bam --remove-indv all_calcaratus_sorted.bam --remove-indv mello_GermSeq_sorted.bam --gzvcf trop_WGS_all_20_samples_all_chrs.vcf.gz --recode --out trop_WGS_no_cal_mello_niger_all_chrs.vcf.gz
```

# Generate the input file in plink format
```bash
FILE=trop_WGS
plink --geno 0.999 --vcf trop_WGS_no_cal_mello_niger_all_chrs.vcf.gz --make-bed --out $FILE --allow-extra-chr --double-id
```
here I had to use --double-id as I had '_' s in my sample names
--geno 0.999 remove all loci where more than 99.9% of genotypes are missing.

This produces a bynch of support files. 

ADMIXTURE does not accept chromosome names that are not human chromosomes. We will thus just exchange the first column by 0

```bash
awk '{$1="0";print $0}' $FILE.bim > $FILE.bim.tmp
mv $FILE.bim.tmp $FILE.bim
```
Now, we are ready to run ADMIXTURE. We will run it with cross-validation (the default is 5-fold CV, for higher, choose e.g. cv=10) and K=2.

Let’s now run it in a for loop with K=2 to K=5 and direct the output into log files

This might take some time to run

Change seed with -s. Or remove this to make it random
-C is termination criteria. Change numbers to meet the maximum number of iterations or 
to stop when the log-likelihood change between iterations falls below that value
```
for i in {2..5}; do  admixture  -C 0.0000001 -s 12345 --cv $FILE.bed $i> log${i}.out; done
```
To identify the best value of k clusters which is the value with lowest cross-validation error, we need to collect the cv errors. Below are three different ways to extract the number of K and the CV error for each corresponding K. Like we said at the start of the course, there are many ways to achieve the same thing in bioinformatics!

use 1 from them

```
awk '/CV/ {print $3,$4}' *out | cut -c 4,7-20 > $FILE.cv.error
grep "CV" *out | awk '{print $3,$4}' | sed -e 's/(//;s/)//;s/://;s/K=//'  > $FILE.cv.error
grep "CV" *out | awk '{print $3,$4}' | cut -c 4,7-20 > $FILE.cv.error
```
To make plotting easier, we can make a file with the individual names in one column and the species names in the second column

# This command extractf file names and pop names for most. But you will have to re check and change them
```
awk '{split($1,name,"_"); print $1,name[2]}' $FILE.nosex > $FILE.list
```
example changed file

```txt
F_Ghana_WZ_BJE4687_combined__sorted.bam Ghana
F_IvoryCoast_xen228_combined__sorted.bam IvoryCoast
F_Nigeria_EUA0331_combined__sorted.bam Nigeria
F_Nigeria_EUA0333_combined__sorted.bam Nigeria
F_SierraLeone_AMNH17272_combined__sorted.bam SierraLeone
F_SierraLeone_AMNH17274_combined__sorted.bam SierraLeone
JBL052_concatscafs_sorted.bam Ref
M_Ghana_WY_BJE4362_combined__sorted.bam Ghana
M_Ghana_ZY_BJE4360_combined__sorted.bam Ghana
M_Nigeria_EUA0334_combined__sorted.bam Nigeria
M_Nigeria_EUA0335_combined__sorted.bam Nigeria
M_SierraLeone_AMNH17271_combined__sorted.bam SierraLeone
M_SierraLeone_AMNH17273_combined__sorted.bam SierraLeone
XT10_WZ_no_adapt._sorted.bam Lab
XT11_WW_trim_no_adapt_scafconcat_sorted.bam Lab
XT1_ZY_no_adapt._sorted.bam Lab
XT7_WY_no_adapt__sorted.bam Lab
all_ROM19161_sorted.bam Liberia
```
Then I downloaded the whole directory to local computer to make it easier to customize
```bash
rsync -axvH --no-g --no-p premacht@beluga.computecanada.ca:/scratch/premacht/admixture_with_admix_2024 .
```




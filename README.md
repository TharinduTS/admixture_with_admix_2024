# admixture_with_admix_2024
This is based on 

https://speciationgenomics.github.io/ADMIXTURE/

I am working in beluga and started by copying vcf file there

Then

# Generate the input file in plink format
```bash
FILE=Pundamilia.RAD
plink --geno 0.999 --vcf no_cal_no_mello_chr7_15mb.vcf.recode.vcf.gz --make-bed --out $FILE --allow-extra-chr --double-id
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

```
for i in {2..5}; do  admixture --cv $FILE.bed $i> log${i}.out; done
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





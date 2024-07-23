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


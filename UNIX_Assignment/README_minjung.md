# UNIX Assignment


## Data Inspection

### Attributes of fang_et_al_genotypes


* only see the header of the file
``` bash
head -n 1 fang_et_al_genotypes.txt
```



* only show the last row of the file
``` bash
tail -n 1 fang_et_al_genotypes.txt
```



* output is number of lines
``` bash
wc -l fang_et_al_genotypes.txt
```



* output shows size of the file
``` bash
ls -lh fang_et_al_genotypes.txt
```



* output shows number of columns
```bash
awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
```



* more robust output that shows number of columns
``` bash
grep -v "^#" fang_et_al_genotypes.txt | awk -F "\t" '{print NF; exit}'
```



* determine whether the file has non-ASCII characters
``` bash
file fang_et_al_genotypes.txt
```



* output shows first column within 10 rows
``` bash
cut -f 1 fang_et_al_genotypes.txt | head -n 10
```



* Identify the problematic character
``` bash
hexdump -c fang_et_al_genotypes.txt
```



**By inspecting this file I learned that:**



* It has 2783 number of lines

* Its size is 11M

* Its number of columns is 986

* It has 100% ASCII text. But it has very long lines

    * That is why the header output was so messy, being seem like a wall of texts.





### Attrubutes of snp_position.txt


* output is a header
``` bash
head -n 1 snp_position.txt
```



* output is a last row of the file
```bash
tail -n 1 snp_position.txt
```



* output is number of lines
```bash
wc -l snp_position.txt
```



* output shows size of the file
```bash
ls -lh snp_position.txt
```



* output shows number of columns
```bash
awk -F "\t" '{print NF; exit}' snp_position.txt
```



* more robust output that shows number of columns
```bash
grep -v "^#" snp_position.txt | awk -F "\t" '{print NF; exit}'
```



* determine whether the file has non-ASCII characters
```bash
file snp_position.txt
```



* output shows first column within 10 rows
```bash
cut -f 1 snp_position.txt | head -n 10
```



* Identify the problematic character
```bash
hexdump -c snp_position.txt
```


**By inspecting this file I learned that:**

* It has 984 number of lines

* Its size is 81K

* Its number of columns is 15

* It has 100% ASCII text. 



## Data processing

### Maize Data

Join needs the same column. But The fang_et_al_genotypes.txt file has samples in rows and SNPs in columns, but my final output needs SNPs in rows.

    

* Include the header and maize groups
```bash
head -n 1 fang_et_al_genotypes.txt > maize_genotypes.txt
```

```bash
grep -E "(ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt >> maize_genotypes.txt
```

 * Transpose the file so snps are rows
 ```bash
awk -f transpose.awk maize_genotypes.txt > transposed_maize.txt
```



#### Prepare the SNP position file 

Snp_id = 1, chromosome = 3, position = 4 in original file

* Extract the targeted columns except the header to avoid it interfering with the sort command later        

```bash
tail -n +2 snp_position.txt | cut -f 1,3,4 | sort -k1,1 > sorted_snp_pos.txt
```



#### Sorting and Joining

Before joining, both files must be sorted by the common column SNP_ID.

* Remove the first three rows of transposed maize (sample_id, JG_OTU, and Group) and sort by snp_id

``` bash
tail -n +4 transposed_maize.txt | sort -k1,1 > sorted_maize.txt
```

* Join the files     

```
join -1 1 -2 1 -t $’\t’ sorted_snp_pos.txt sorted_maize.txt > joined_maize.txt
```




#### Generating the 20 chromosome files

* Install bioawk program        

```bash
module load bioawk
```

* Files with increasing position and missing data = ?

```bash
for i in {1..10}; do bioawk -c hdr -v chr="$i" '$2 == chr' joined_maize.txt | sort -k3,3n > maize_chr${i}_inc.txt; done
```

* Files with decreasing position and missing data = -      

```bash
for i in {1..10}; do bioawk -c hdr -v chr="$i" '$2 == chr' joined_maize.txt | sed 's/?/-/g' | sort -k3,3nr > maize_chr${i}_dec.txt; done
```



#### Handling unknown and multiple positions

* Unknown positions        

```bash
grep “unknown” joined_maize.txt > maize_unknown_pos.txt
```

* Multiple positions        

```bash
grep “multiple” joined_maize.txt > maize_multiple_pos.txt
```



#### Verification 

```bash
wc -l
```





### Teosinte Data

Join needs the same column. But The fang_et_al_genotypes.txt file has samples in rows and SNPs in columns, but my final output needs SNPs in rows.

* Include the header and maize groups         

``` bash
head -n 1 fang_et_al_genotypes.txt > teosinte_genotypes.txt
```


```bash
grep -E "(ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt >> teosinte_genotypes.txt
```

* Transpose the file so snps are rows     

```bash
awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte.txt
```



#### Sorting and Joining

Before joining, both files must be sorted by the common column SNP_ID.

* Extra care easily ignored dots     

```bash
export LC_ALL=C
```

* Remove the first three rows of transposed teosinte (Sample_ID, JG_OTU, and Group) and sort by SNP_ID     

```bash
tail -n +4 transposed_teosinte.txt | sort -k1,1 > sorted_teosinte.txt
```

* Join the files, which are tab-delimited     

```bash
join -1 1 -2 1 -t $’\t’ sorted_snp_pos.txt sorted_teosinte.txt > joined_teosinte.txt
```



#### Generating the 20 chromosome files

* Install bioawk program     

```bash
module load bioawk
```

* Files with increasing position and missing data = ?    

```bash
for i in {1..10}; do bioawk -c hdr -v chr="$i" '$2 == chr' joined_teosinte.txt | sort -k3,3n > teosinte_chr${i}_inc.txt; done
```

* Files with decreasing position and missing data = -    

```bash
for i in {1..10}; do bioawk -c hdr -v chr="$i" '$2 == chr' joined_teosinte.txt | sed 's/?/-/g' | sort -k3,3nr > teosinte_chr${i}_dec.txt; done
```



#### Handling unknown and multiple positions

* Unknown positions     

```bash
grep “unknown” joined_teosinte.txt > teosinte_unknown_pos.txt
```

* Multiple positions     

```bash
grep “multiple” joined_teosinte.txt > teosinte_multiple_pos.txt
```



#### Verification 
```bash
wc -l
```


[comment]: <> (Create chewbbaca-tutorial git)

All information about NCBI genomes used in this example is on the .tsv file inside the genomes folder, view file [here](https://github.com/mickaelsilva/chewBBACA_tutorial/blob/master/genomes/NCBI_genomes_proks.Sagalactiae_allGenomes.2016_08_03.tsv).

## Schema creation 
was performed using the **32** _Streptococcus agalactiae_ complete genomes (32 genomes with a level of assembly classified as complete genome or chromossome)  available at NCBI,
 resulting in a wgMLST schema of **3128 loci**.

`chewBBACA.py CreateSchema -i complete_genomes/ --cpu 6 -o schema_seed -t "Streptococcus agalactiae"`

Run on intel core i7 with:

    Starting Script at : 13:50:57-09/02/2017
    Finished Script at : 14:07:09-09/02/2017

## Alleles were called
for the **32** genomes using the wgMLST schema and results were analysed. 

```chewBBACA.py Allelecall -i listgenomes.txt -g listgenes.txt -o results --cpu 6 -t "Streptococcus agalactiae"```

Run on intel core i7 with:
    
    Starting Script at : 11:49:53-04/07/2017
    Finished Script at : 12:12:05-04/07/2017
    number of genomes: 32
    number of loci: 3128
    used this number of cpus: 6
    used a bsr of : 0.6

## Paralog detection 
 
returned a set of 24 loci that were removed from further analysis.

`chewBBACA.py RemoveGenes -i results_alleles.txt -g RepeatedLoci.txt -o alleleCallMatrix_cg.tsv`


A set of **1136** loci were found to be present in all the analyzed complete genomes, while **1264** loci were present in at least 95%.

`chewBBACA.py TestGenomeQuality -i alleleCallMatrix_cg.tsv -n 13 -t 200 -s 5`

![Genome quality testing of complete genomes](http://i.imgur.com/Zh6GRk9.png)
[larger image fig 1](http://i.imgur.com/Zh6GRk9.png)


## Enriching the dataset analysis
with other assemblies submitted as **Streptococcus Agalactiae** on NCBI were downloaded ( 03-08-2016, downloadable zip file [here](https://drive.google.com/file/d/0Bw6VuoagsdhmaWEtR25fODlJTEk/view?usp=sharing)) 
and analyzed with [MLST](https://github.com/tseemann/mlst) in order to exclude possible mislabeled 
samples as Streptococcus Agalactiae. A total of **682 genomes** were downloaded being 2 (GCA_000323065.2_ASM32306v2 and GCA_001017915.1_ASM101791v1) detected 
as being a different species/contamination and removed from the analysis. 
Allele call was performed on the remaining 680 genomes using the **1264 loci** for schema validation. Paralog detection found no paralog loci.


```chewBBACA.py Allelecall -i .genomes/ -g listgenes_core.txt -o results --cpu 6 -t "Streptococcus agalactiae"```

Run on a slurm based HPC :

    Starting Script at : 08:48:16-05/07/2017
    Finished Script at : 09:38:34-05/07/2017
    number of genomes: 680
    number of loci: 1264
    used this number of cpus: 20
    used a bsr of : 0.6

Since our complete genomes allele call was performed with the wgMLST schema we need to remove the loci that constitute the auxiliary genome, in order to concatenate the matrixes with the same loci.

```chewBBACA.py RemoveGenes -i alleleCallMatrix.tsv -g listgenes_core.txt -o cg_completegenomes --inverse```

This command removes all loci that are not present int listgenes_core. The output cg_completegenomes.tsv file can now be concatenated with the result allele call of the 680 genomes.

While the presence of locus in 95% of genomes remains constant at around **1200** loci,considering all 
or most of the genomes (90%<x<100%), the number of loci present in all genomes is very low (34) and presents some variation when some specific genomes are removed from the analysis.

`chewBBACA.py TestGenomeQuality -i results_alleles.tsv -n 13 -t 300 -s 5`

![Genome quality testing of all genomes](http://i.imgur.com/j4u22ZE.png)
[larger image here fig 2](http://i.imgur.com/j4u22ZE.png)

We selected the results at threshold 25 for further analysis as it presented a significant
 loci presence in 95% of genomes change (+50 loci) and an acceptable loci 
 presence in all considered genomes (650 genomes/437 loci). 

`chewBBACA.py ExtractCgMLST -i results_alleles.tsv -o cgMLST_25 -g removedGenomes_25.txt`

## Minimum Spanning Tree
cgMLST_25.tsv was uploaded to Phyloviz online and can be accessed [here](https://online.phyloviz.net/main/dataset/share/cfab1610a3ca3a80cf9c139e436ce741fc5fa29dcc5aeb3988025491d4434143fc72f6284df3ac8256322fb3e59e5284e231585daf349a2e37d6ece96f4f746d8a2eee0b82f6a609583967bb011543003fa881128b83bfeacf02140fb441)


## Genome Quality analysis
Since no quality control was previously performed, it is possible that some assemblies 
here included were of lower quality. A general analysis on the assemblies show a N50 
variation that ranges from 8055 to over 2.2M, while the number of contigs ranges between 
1 and 553 per sample. These results made us more suspicious on how the quality of the 
genomes may had affected the allele call results and consequently the severe drop on 
locus present in all genomes.  

As stated previously, in order to obtain a considerable comparable cgMLST profile, 
some genomes had to be removed (62) since they presented more missing data than others. 
In order to assess the possible reason of their poor allele call performance, two plots 
were built. The removed genomes were then highlighted and dashed lines linking the same 
genomes annotations. 

The first plot representing the total number of bp on contigs with a size over 
10k bp and the N50 of all assemblies, sorted by decreasing values

![Genome Analysis](http://i.imgur.com/I0fNqtd.png)
[larger image fig 3](http://i.imgur.com/I0fNqtd.png)

The second plot representing the total number of contigs and the number of 
contigs over 10k bp

![Genome Analysis 2](http://i.imgur.com/fabxi0Z.png)
[larger image](http://i.imgur.com/fabxi0Z.png)

At first sight, most of the removed genomes (57) were located on the lower number of 
bp/N50 (fig.3) and the higher number of contigs (fig.4)

The remaining 5 genomes were individually checked :
    
1. **GCA_000186445.1** [here](https://www.ncbi.nlm.nih.gov/assembly/GCA_000186445.1) - 21 contigs but only 1 is above 10k (Scaffold with lot of N ,134 real contigs)
2. **GCA_000221325.2** [here](https://www.ncbi.nlm.nih.gov/assembly/GCA_000221325.2)- NCBI curated it out of RefSeq for having a genome length too large
3. **GCA_000427055.1** [here](https://www.ncbi.nlm.nih.gov/assembly/GCA_000427055.1)- NCBI curated it out of RefSeq for having many frameshifted proteins
4. **GCA_000289455.1** [here](https://www.ncbi.nlm.nih.gov/assembly/GCA_000289455.1)- No ST found...
5. **GCA_000288835.1** [here](https://www.ncbi.nlm.nih.gov/assembly/GCA_000288835.1)- NCBI curated it out of RefSeq for having many frameshifted proteins


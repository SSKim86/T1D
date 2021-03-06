# Disease SNP prioritization

This is an protocol for prioritization of SNPs associated certain phenotype/disease. Here is a study for prioritization of SNPs associated with Type 1 diabetes. You can follow the below analysis steps.



## 1. Seed SNPs preparation for type 1 diabetes (T1D)

### > gwas.r

To download **GWAS Catalog data** (MacArthur et al, 2017, Nucleic Acids Research, pmid 27899670), you can [search certain disease](https://www.ebi.ac.uk/gwas/). In this study, we downloaded [SNP-sets for type 1 diabetes](https://www.ebi.ac.uk/gwas/efotraits/EFO_0001359). Then you can run R code file for filtering the GWAS Catalog data as below `CMD` command line:

- Bellow functions are run under the windows or linux command console environment.
- Instead of `[ ]`, you have to put the arguments `file path` or `value` by the options.
- Usage: `Rscript gwas.r [GWAS_file_path] [p-value_criteria]`

```cmd
Rscript T1D_gwas.r db/GWAS_EFO0001359.tsv 5e-08
```

### > ldlink_dn.py / link_filt.r

To download **LDlink data** (version 3.3.0 12/24/2018) (Machiela et al, 2015, Bioinformatics, pmid 26139635), you can run `T1D_ldlink.py` as below `CMD` command line:

- To run the code, you need list of SNP RS IDs of dbSNP database as txt file
- Usage: `python ldlink.py [SNP_file_path.txt]`

```CMD
python ldlink_dn.py data/gwas_5e-08_129.tsv
```

> ...
> 129/129 = rs11580078
>   status_code = 200
>   line number = 950
>   file saved = db/ldlink/rs11580078.tsv
>
> Download process completed.
> Job time= 00:42:49

To filter the LDlink data by statistical criteria, both r<sup>2</sup> >0.6 and D'=1, you can run `T1D_ldlink.r` as below `CMD` command line:

- Usage: `Rscript ldlink.r [SNP_file_path.txt] [LDlink_data_folder_path] [LDlink_filter_option]`
- The `LDlink_filter_option` is a mandatory. Choose one of the following option numbers.
  1. `r2>0.6 or Dprime=1`
  2. `r2>0.6`
  3. `Dprime=1`
  4. `r2>0.6 and Dprime=1`

```CMD
Rscript ldlink_filt.r data/gwas_5e-08_129.tsv db/ldlink 2
```

> Input SNP list number = 129
>
> Error in read.table(as.character(snptb[i, 2]), header = T) :
>   more columns than column names
> In addition: Warning message:
> In read.table(as.character(snptb[i, 2]), header = T) :
>   incomplete final line found by readTableHeader on 'db/ldlink/rs75793288.tsv'
> NULL
> Filtering option, r2 > 0.6 was chosen.
> ::Expluded no rsid elements = 31
>
> 1/3. Numbers of SNPs
> SNP Tier1 = 129
> SNP Tier2 = 5116
> SNP seed  = 5245
>
> 2/3. Generation of a result TSV file
> File write: data/seedSNP_5245_ldlink.tsv
>
> 3/3. Generation of a result BED file
> Table, rows= 5245 cols= 4
> File write: data/seedSNP_5245.bed
> Job done for 7.9 sec

* `data/seedSNP_5245_ldlink.tsv` -> **Supplementary Table 1** and **Supplementary Table 2**

### Q1. Generation of private SNP list (rsids) to BED file format?

To use bedtools later, you have to prepare SNP list as [bed format](https://genome.ucsc.edu/FAQ/FAQformat.html). If you have simple dbSNP rsid list, you can run `src/biomart_snp.r` for generate bed file. But you should check `NA` values and fill it manually.

```CMD
#Rscript src/biomart_snp.r [rsid_list_file_path]
Rscript src/biomart_snp.r data/seedSNP_5245_biomart.txt
```

> Input contents, rows= 5244 cols= 1
> Table, rows= 10280 cols= 4
>
> Job done for 12.7 sec

**[IMPORTANT]** Before you move to next step, we make sure that your `seedSNP_#.bed` file has no NA values.



## 2. RoadMap data download and filter

The [RoadMAP project](https://egg2.wustl.edu/roadmap/web_portal/imputed.html) provides epigenome annotations such as [12-mark/127-reference epigenome/25-state Imputation Based Chromatin State Model](https://egg2.wustl.edu/roadmap/data/byFileType/chromhmmSegmentations/ChmmModels/imputed12marks/jointModel/final/) by using ChromHMM algorithm (Ernst and Kellis, 2012, Nature Methods, pmid 22373907). We downloaded the 127 files by their [cell types](https://github.com/mdozmorov/genomerunner_web/wiki/Roadmap-cell-types) (e.g., `E001_25_imputed12marks_hg38lift_dense.bed.gz` and etc) using R code (`T1D_roadmap.r`). And then we filtered the data by [annotation code](https://egg2.wustl.edu/roadmap/web_portal/imputed.html) (see db/[roadmap] ) including 13_EnhA1, 14_EnhA2, 15_EnhAF, 16_EnhW1, 17_EnhW2, 18_EnhAc.

To download RoadMap data, you need to install `AnnotationHub` and `rtracklayer` in `BiocManaer` as below R code:

```R
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install("AnnotationHub", version = "3.8")
BiocManager::install("rtracklayer", version = "3.8")
```

### > roadmap_dn.r

To download **RoadMap data**, you can run `roadmap_dn.r` as below `CMD` command line:

- The 127 cell type-specific RoadMap BED files will download at `db/roadmap` folder
- This process takes about ~30 min that depends on your download speed.

```CMD
Rscript roadmap_dn.r
```

### > roadmap_filt.r

To filter the RoadMap data by **Enhancers**, you can run `roadmap_filt.r` as below `CMD` command line:

- The result file would be saved as `data/roadmap_enh.bed`
- This process takes ~3 min that depends on your computer processor speed.

```CMD
Rscript roadmap_filt.r
```

### Q2. If you have memory problem..

When running the `roadmap_filt.r` function, it stop with not enough memory error, You can use `roadmap_filt_dtr.r` function for limited memory usage (~3.8 GB).

```CMD
Rscript roadmap_filt_dtr.r
```

### $ bedtools merge/ bedtools closest

To avoid multiple count of enhancers as well as to reduce file size and to achieve faster process, merge RoadMap enhancer information using a `BASH` tool `bedtools`. Here is the `BASH` pipeline for `bedtools sort` and `bedtools merge`. Then, to identify T1D SNPs occupied in RoadMap enhancers, you can use `BASH` tool `bedtools intersect` as below code:

- Compressed file size of `roadmap_enh.bed.gz` is >139 MB.
- Compressed file size of `roadmap_enh_merer.bed.gz` is about 3.7 MB.
- Removing NA values, `data/seedSNP_5245_bm.bed` file is updated version from the `data/seedSNP_5245.bed` file.

```SHELL
bedtools sort -i db/roadmap_enh.bed | bedtools merge -i stdin -c 1 -o count > db/roadmap_enh_merge.bed
bedtools sort -i data/seedSNP_5245_bm.bed | bedtools closest -d -a stdin -b db/roadmap_enh_merge.bed > data/roadmap_dist.tsv
```

### > src/bedtools_closest.r

To prioritize RoadMap enhancer occupied SNPs, you can run `src/bedtools_closestroadmap.r` as below `CMD` command line:

- `data/roadmap_dist_df.tsv` file is obtained that is for enhancer annotated file .
- `data/snp_484_roadmap_dist.bed` file is obtained that is for `BED` format file for USCS browser.
- Usage: `Rscript src/bedtools_closest.r [bedtools_closest_result_file_path] [double_line_result]`

```CMD
Rscript src/bedtools_closest.r data/roadmap_dist.tsv False
```

> Row number = 5245
> Enhancer occupied by SNPs = 589
> SNPs in RoadMap enhancers = 1688
>
> File write: data/roadmap_dist_df.tsv
> File write: data/snp_1688_roadmap_dist.bed
>
> Job done for 0.2 sec

* `data/roadmap_dist_df.tsv` -> **Supplementary Table 2**

### Q3. How about just use not merged roadmap_enh.bed file?

Instead of merge file, when you use original `db/roadmap_enh.bed` file, you can find a lot of duplicated enhancers regions.

```SHELL
bedtools sort -i db/roadmap_enh.bed | bedtools closest -d -a data/seedSNP_1817.bed -b stdin > data/roadmap_dist2.tsv
```



## 3. ENCODE ChIP-seq data download and filter

The **ENCODE ChIP-seq** for regulatory transcription factor binding site (Reg-TFBS) cluster data can downloaded <u>wgEncodeRegTfbsClusteredV3</u> data from [UCSC FTP](http://hgdownload.cse.ucsc.edu/goldenpath/hg19/encodeDCC/wgEncodeRegTfbsClustered/) (68 MB) or [bioconductor `data("wgEncodeTfbsV3")`](https://www.bioconductor.org/packages/devel/bioc/vignettes/ChIPpeakAnno/inst/doc/ChIPpeakAnno.html). Here, we assume having downloaded UCSC FTP file `wgEncodeRegTfbsClusteredV3.bed.gz` (81 MB).

```CMD
Rsciprt encode_dn.r
```

> Directory generated: db/encode/
> trying URL 'http://hgdownload.cse.ucsc.edu/goldenpath/hg19/encodeDCC/wgEncodeRegTfbsClustered/wgEncodeRegTfbsClusteredV3.bed.gz'
> Content type 'application/x-gzip' length 84986946 bytes (81.0 MB)
>
> downloaded 81.0 MB
>
> db/encode/wgEncodeRegTfbsClusteredV3.bed.gz
>
> Job done for 1.8 min

### $ bedtools merge | bedtools closest

To identify TFBS occupied SNPs, you can use `bedtools merge` and `bedtools closest` as following code:

- Merging the ENCODE TFBS data give you benefits such as avoiding multiple count of enhancers as well as reducing file size and achieving faster process

```SHELL
bedtools merge -i db/wgEncodeRegTfbsClusteredV3.bed.gz -c 1 -o count > db/encode_tfbs_merge.bed
bedtools sort -i data/seedSNP_5245_bm.bed | bedtools closest -d -a stdin -b db/encode_tfbs_merge.bed > data/encode_dist.tsv
```

### > src/bedtools_closest.r

To prioritize ENCODE Reg-TFBS occupied SNPs, you can run `src/bedtools_closestroadmap.r` as below `CMD` command line:

- `data/roadmap_dist_df.tsv` file is obtained that is for enhancer annotated file .
- `data/snp_enh_484.bed` file is obtained that is for `BED` format file for USCS browser.
- Usage: `Rscript src/bedtools_closest_roadmap.r [bedtools_closest_result_file_path] [double_line_result]`

```CMD
Rscript src/bedtools_closest.r data/encode_dist.tsv False
```

> Row number = 5245
> Enhancer occupied by SNPs = 653
> SNPs in RoadMap enhancers = 1253
>
> File write: data/encode_dist_df.tsv
> File write: data/snp_1253_encode_dist.bed
>
> Job done for 0.2 sec

* `data/encode_dist_df.tsv` -> **Supplementary Table 2**



## 4. Regulome DB data download and filter

The [**RegulomeDB**](http://www.regulomedb.org/index) provides [category scores for SNPs by evidences](http://www.regulomedb.org/help) (see `Regulome score.txt`), including eQTL, TF binding, matched TF motif, matched DNase Footprint, and DNase peak. In this study, we stringently filtered and used high-score (`≥ 2b`) SNPs for our study. Before you start, you can download the files from [Regulome DB download page](http://www.regulomedb.org/downloads). To make this process faster, you can convert the downloaded files to RDS format.

- `RegulomeDB.dbSNP132.Category1.txt.gz` (2 MB)
- `RegulomeDB.dbSNP132.Category2.txt.gz` (39.3 MB)
- Or you can download total dataset: `RegulomeDB.dbSNP141.txt.gz` (2.8 GB)

```CMD
Rscript regulome_dn.r
```

> In dir.create(file.path(dir)) : 'db\regulome' already exists
> trying URL 'http://www.regulomedb.org/downloads/RegulomeDB.dbSNP132.Category1.txt.gz'
> Content type 'application/gzip' length 2096454 bytes (2.0 MB)
>
> downloaded 2.0 MB
>
> trying URL 'http://www.regulomedb.org/downloads/RegulomeDB.dbSNP132.Category2.txt.gz'
> Content type 'application/gzip' length 41253483 bytes (39.3 MB)
>
> downloaded 39.3 MB
>
> Job process: 52.2 sec
> File write: db/regulome/RegulomeDB.dbSNP132.Category1.txt.gz.rds
> File write: db/regulome/RegulomeDB.dbSNP132.Category2.txt.gz.rds
> Job done for 1.2 min

Here we converted the download files to RDS format files to achieve fast loading speed. Use the RegulomeDB RDS files, you can filter and analyze the dataset by using `regulome.r` as following command line:

```CMD
Rscript regulome.r data/seedSNP_5245_bm.bed db/RegulomeDB.dbSNP132.Category1.txt.rds db/RegulomeDB.dbSNP132.Category2.txt.rds
```

> Input SNPs number = 5245
> Input regulome files = 2
>
> Read input file 1: db/RegulomeDB.dbSNP132.Category1.txt.rds
> Table row = 39432, col = 5
> Job process: 0.3 sec
>
> Read input file 2: db/RegulomeDB.dbSNP132.Category2.txt.rds
> Table row = 407796, col = 5
> Job process: 6.1 sec
>
> Regulome score >=2b, SNPs = 430528
> Functional motifs (1f_only-2b) = 395823
>   - Regulome >=2b SNPs = 301
>   - SNPs with functional motifs (1f_only-2b) = 138
>
> File write: data/regulome_301.tsv
> File write: data/snp_301_regulome2b.bed
> Job done: 2019-08-15 17:43:22 for 6.4 sec

The result files are save at `data/` folder:

- `data/regulome_301.tsv` -> **Supplementary Table 2**
- `data/snp_301_regulome2b.bed`



### Venn analysis to identify core SNPs

Summary for SNPs with RoadMap annotation, ENCODE ChIP-seq, and RegulomeDB. This R code for Venn analysis uses **Bioconductor** `limma` R package. The installation of the `limma` package as below:

```R
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install("limma", version = "3.8")
```

To prioritize the SNPs, you can run `venn.r` as below `CMD` command line with these files:

- `data/snp_1688_roadmap_dist.bed`
- `data/snp_1253_encode_dist.bed`
- `data/snp_301_regulome2b.bed`

### > venn.r

```CMD
Rscript venn.r data/snp_1688_roadmap_dist.bed data/snp_1253_encode_dist.bed data/snp_301_regulome2b.bed
```

> [1] "snp_1688_roadmap_dist"
> [1] "snp_1253_encode_dist"
> [1] "snp_301_regulome2b"
> File write: data/venn.tsv
> File write: data/vennCounts.tsv
>
> Euler fit is done.
> Figure draw: fig/euler_snp_1688_roadmap_dist_snp_1253_encode_dist.png
> File write: data/snp_79_core.bed
>
> Job done for 0.5 sec

The result files are generated as below:

- `venn_tfbs.tsv`: binary SNP overlap table
- `vennCounts_tfbs.tsv`: overlapped SNP numbers
- `snp_79_core.bed`

The result figure is generated as below:

![Venn analysis of 1817 SNPs](./fig/euler_snp_1688_roadmap_dist_snp_1253_encode_dist.png)

## 5. GTEx eQTL data download and filter

The [Genotype-Tissue Expression (GTEx)](https://gtexportal.org/home/) project is a public resource to study tissue-specific gene expression and their regulation by SNPs. GTEx version 7 includes 11,688 samples, 53 tissues and 714 donors. You can download [GTEx eQTL data](https://gtexportal.org/home/datasets) `GTEx_Analysis_v7_eQTL.tar.gz` (915 MB) and filter by statistical criteria `p < 3e-04`. The `GTEx_Analysis_v7_eQTL.tar.gz` compressed file includes:

- 48 files with `db/GTEx_Analysis_v7_eQTL/*.egenes.txt` extensions
- 48 files with `db/GTEx_Analysis_v7_eQTL/*.signif_variant_gene_pairs.txt` extensions

And we need SNP annotations to achieve Rsid for GTEx ids.

- `db/GTEx_Analysis_2016-01-15_v7_WholeGenomeSeq_635Ind_PASS_AB02_GQ20_HETX_MISS15_PLINKQC.lookup_table.txt.gz` (440 MB)
- [Nominal p-values](https://gtexportal.org/home/documentationPage) from GTEx data were generated for each variant-gene pair by testing the alternative hypothesis that the slope of a linear regression model between genotype and expression deviates from 0.

### > gtex_dn.r [and] gtex_filt.r

```CMD
Rscript gtex_dn.r
```

> (1/2) Download eQTL data
> trying URL 'https://storage.googleapis.com/gtex_analysis_v7/single_tissue_eqtl_data/GTEx_Analysis_v7_eQTL.tar.gz'
> Content type 'application/x-tar' length 959746583 bytes (915.3 MB)
>
> downloaded 915.3 MB
>
> File write: db/gtex_files.txt
>
> (2/2) Download SNPid annotation file
> trying URL 'https://storage.googleapis.com/gtex_analysis_v7/reference/GTEx_Analysis_2016-01-15_v7_WholeGenomeSeq_635Ind_PASS_AB02_GQ20_HETX_MISS15_PLINKQC.lookup_table.txt.gz'
> Content type 'application/gzip' length 461926948 bytes (440.5 MB)
>
> downloaded 440.5 MB
>
> Job done for 1.2 min

`> 50 min` The downloaded files were converted to RDS files.

```CMD
Rscript gtex_rds.r
```

>  - Loading GTEx BED files
>  - File reading...
>     (1/48) Adipose_Subcutaneous
>     ...
>   (48/48) Whole_Blood
> NULL
>  gte.df.pval_nominal
>  Min.   :0.000e+00
>  1st Qu.:0.000e+00
>  Median :1.111e-07
>  Mean   :7.398e-06
>  3rd Qu.:5.219e-06
>  Max.   :8.005e-04
>  - GTEx table, rows= 36781356 cols= 13
>  - BED file read complete. Job process: 9.3 min
>  - Loading annotation file - Annotation file read complete. Job process: 35.5 min
>  - Annotation file, rows= 40738696 cols= 7
>  - GTEx annotation, rows= 36781356 cols= 9
>
> Job process: 50.7 min
>
>  - Saving as RDS file..
>
> Job process: 52 min
>
> Job done for 52 min

To filter the GTEx data by p <5e-08, I executed following code:

```CMD
Rscript gtex_filt.r 5e-08
```

> p-value threshold = 5e-08
>
> (1/3) Loading GTEx RDS file
>  - GTEx table, rows= 36781356 cols= 9
>  - BED file read complete. Job process: 53.3 sec
>
> (2/3) Filtering by nominal p-value
>  gte.sig.pval_nominal
>  Min.   :0.000e+00
>  1st Qu.:0.000e+00
>  Median :3.120e-12
>  Mean   :3.897e-09
>  3rd Qu.:1.429e-09
>  Max.   :5.000e-08
>  - GTEx significant, rows= 17113536 cols= 9
>  - Job process: 1.1 min
>
> File write: db/gtex_signif.tsv
> Job done for 3.8 min

The result file size are huge and the process takes long time (~50 min)

- `gtex_signif_5e-8.tsv.rds` (322 MB)
- This file was compressed by `zip` as three separated <100 MB files.
  1. `gtex_signif_5e-8.tsv.zip`
  2. `gtex_signif_5e-8.tsv.z01`
  3. `gtex_signif_5e-8.tsv.gz.z02`

To identify T1D SNPs 

```CMD
Rscript gtex.r data/seedSNP_5245_bm.bed db/gtex_signif_5e-8.tsv.rds
```

> Input SNPs number = 5,245
>
> (1/3) Loading GTEx significant file
>   - gtex_signif_5e-8.tsv.rds: rows= 17,113,536 cols= 9
>   - Job process: 29.3 sec
>
> (2/3) eQTL SNP filteration
>   - Overlapped table, rows= 137,762 cols= 9
>   - eQTL SNPs = 2,676
>   - Associated genes = 192
>
> File write: data/gtex_5e-08_2676.tsv
>
> (3/3) eQTL SNP BED file generation
>   - GTEx SNP BED, rows= 2,676 cols= 4
>   - eQTL SNPs = 2,676
>
> File write: data/snp_2676_gtex.bed
>
> Job done for 33.8 sec

The result files of criteria 5e-08 are here:

- `gtex_5e-08_2676.tsv` -> **Supplementary Table 3**
- `snp_2676_gtex.bed`

### Venn analysis and overlap SNPs

To prioritize the eQTL SNPs among the 26 high-probability causal enhancer SNPs, you can run `venn.r` as below `CMD` command line with these files:

- `data/snp_570_roadmap_encode.bed` - Enhancer occupied SNP list **<- manually generated**
- `data/snp_79_core.bed` - High-probability causal enhancer SNP list
- `data/snp_2676_gtex.bed` - eQTL SNP list

### > venn.r

```CMD
Rscript venn.r data/snp_570_roadmap_encode.bed data/snp_79_core.bed data/snp_2676_gtex.bed
```

The result files are generated as below:

- `venn.tsv` > `venn_gtex.tsv`: binary SNP overlap table
- `vennCounts.tsv` > `vennCounts_gtex.tsv`: overlapped SNP numbers
- `snp_79_core.bed` > `snp_79_core_tfbs.bed`: SNP `BED` format file

The result figure is generated as below:

![](./fig/euler_snp_570_roadmap_encode_snp_79_core.png)



## 5-1. Nearest gene approach

### downloading Ensembl gene location data

To identify nearest genes from the eQTL SNPs, firstly you need to download gene location data from Ensembl database biomart (version=Grch37). 

```CMD
Rscript src/biomart_gene.r
```

>  - Ensembl table, rows= 63677 cols= 5
>  - File write: db/ensembl_gene_ann.tsv
>
>  - Filter result, rows= 57736 cols= 5
>  - File write: db/ensembl_gene.bed
>
> Job done for 17.1 sec

### $ bedtools closest

To identify nearest genes from the eQTL SNPs, you can use `bedtools merge` and `bedtools closest` as following `BASH` codes:

```SHELL
bedtools sort -i db/ensembl_gene.bed | bedtools closest -d -a data/seedSNP_5245_bm.bed -b stdin > data/seedSNP_nearest.tsv
```

```SHELL
bedtools sort -i db/ensembl_gene.bed | bedtools closest -d -a data/snp_2676_gtex.bed -b stdin > data/gtex_nearest.tsv
```

### > src/bedtools_closest.r

To prioritize RoadMap enhancer occupied SNPs, you can run `src/bedtools_closestroadmap.r` as below `CMD` command line:

- Usage: `Rscript src/bedtools_closest_gtex.r [bedtools_closest_result_file_path]`

```CMD
Rscript src/bedtools_closest_gtex.r data/seedSNP_nearest.tsv
```
> Row number = 5861
> Input SNPs = 5245
> Nearest genes = 374
>
> File write: data/seedSNP_nearest_df.tsv
>
> Job done for 0.4 sec


```CMD
Rscript src/bedtools_closest_gtex.r data/gtex_nearest.tsv
```

> Row number = 2909
> Input SNPs = 2676
> Nearest genes = 308
>
> File write: data/gtex_nearest_df.tsv
>
> Job done for 0.2 sec

* `data/gtex_nearest_df.tsv` -> **Supplementary Table 4**

### > src/gtex_overlap.r

To identify the eQTL SNPs occupied on TFBS binding enhancers, you can run `src/gtex_overlap.r` as below `CMD` command line:

```CMD
Rscript gtex_overlap.r data/snp_570_roadmap_encode.bed data/gtex_5e-08_2676.tsv data/gtex_nearest_df.tsv
```

> (1/2) Read files..
>   - data/snp_570_roadmap_encode.bed, rows= 570 cols= 4
>   - data/gtex_5e-08_2676.tsv, rows= 137762 cols= 9
>   - data/gtex_nearest_df.tsv, rows= 1952 cols= 7
>
> SNPs= 2676 Genes= 192 (Nearest= 91)
>
> (2/2) Overlap the two files..
>   - TFBS overlap, rows= 41441 cols= 10
>
> SNPs= 348 Genes= 140 (Nearest= 63)
>
> File write: data/snp_348_gtex_enh.bed



## 6. lncRNASNP2 data download and filter

Human SNPs located in long non-coding RNAs (lncRNAs) are archived in [**lncRNASNP2 database**](http://bioinfo.life.hust.edu.cn/lncRNASNP#!/). You can download these data at the [download page](http://bioinfo.life.hust.edu.cn/lncRNASNP#!/download):

- `lncRNASNP2_snplist.txt.gz` - **SNP list** includes the list of human SNPs in lncRNASNP database.
- `lncrnas.txt.gz` - **lncRNA list** includes the list of human lncRNAs in lncRNASNP database.
- `lncrna-diseases_experiment.txt.gz` - **Experimental validated lncRNA-associated diseases** includes all experiment validated lncRNA-associated diseases.
- `Rscript lncrnasnp.r [SNP_BED_file_path] [lncRNAsnp2_SNP_list_file_path] [lncRNAsnp2_lncRNA_list_file_path] [lncRNAsnp2_diseases_list_file_path]`

```CMD
Rscript lncrnasnp_dn.r
```

> 1: package 'data.table' was built under R version 3.5.2
>
> 2: package 'GenomeInfoDb' was built under R version 3.5.2
>
> trying URL 'http://bioinfo.life.hust.edu.cn/static/lncRNASNP2/downloads/snps_mod.txt'
>
> Content type 'text/plain; charset=GBK' length 477785336 bytes (455.7 MB)
>
> downloaded 455.7 MB
>
> trying URL 'http://bioinfo.life.hust.edu.cn/static/lncRNASNP2/downloads/lncrnas.txt'
>
> Content type 'text/plain; charset=GBK' length 7005411 bytes (6.7 MB)
>
> downloaded 6.7 MB
>
> trying URL 'http://bioinfo.life.hust.edu.cn/static/lncRNASNP2/downloads/lncRNA_associated_disease_experiment.txt'
>
> Content type 'text/plain; charset=GBK' length 31542 bytes (30 KB)
>
> downloaded 30 KB
>
> Job process: 1.4 min
>
> File write: db/lncRNASNP2_snplist.txt.rds
>
> File write: db/lncrnas.txt.rds
>
> File write: db/lncrna-diseases_experiment.txt.rds
>
> Job done for 2.2 min

To identify lncRNA overlapped longevity SNPs:

```CMD
Rscript lncrnasnp.r data/seedSNP_5245_bm.bed db/lncRNASNP2_snplist.txt.rds db/lncrnas.txt.rds db/lncrna-diseases_experiment.txt.rds
```

> (1/3) Read files..
>   - data/seedSNP_5245_bm.bed; Job process: 0.1 sec
>   - db/lncRNASNP2_snplist.txt.rds; Job process: 17.3 sec
>   - db/lncrnas.txt.rds; Job process: 17.5 sec
>   - db/lncrna-diseases_experiment.txt.rds; Job process: 17.5 sec
>   
> | path                                  |     nrow | ncol |
> | :------------------------------------ | -------: | ---: |
> | data/seedSNP_5245_bm.bed              |     5245 |    4 |
> | db/lncRNASNP2_snplist.txt.rds         | 10205295 |    2 |
> | db/lncrnas.txt.rds                    |   141271 |    4 |
> | db/lncrna-diseases_experiment.txt.rds |      753 |    3 |
>   - Job process: 17.7 sec
>
> (2/3) Overlapping lncRNA to my SNP list and binding annotation..
> 
> | lncRNA | SNPs |
> | -----: | ---: |
> |    147 |  261 |
> File write: data/snp_261_lncrnasnp.bed
>
> (3/3) Annotating SNPs in lncRNAs
>
> File write: data/lncrnasnp_261.tsv
>
> Job done for 22.8 sec

- `data/snp_261_lncrnasnp.bed` - 261 SNPs `BED` file
- `data/lncrnasnp_261.tsv` -> **Supplementary Table 5**

### > venn.r

```CMD
Rscript venn.r data/snp_79_core_tfbs.bed data/snp_348_gtex_enh.bed data/snp_261_lncrnasnp.bed
```

> [1] "snp_79_core_tfbs"
> [1] "snp_348_gtex_enh"
> [1] "snp_261_lncrnasnp"
> File write: data/venn.tsv
> File write: data/vennCounts.tsv
>
> Euler fit is done.
> Figure draw: fig/euler_snp_79_core_snp_348_gtex_enh.png
> File write: data/snp_7_core.bed
>
> Job done for 0.3 sec

- `venn.tsv` -> `venn_lncrnasnp.tsv`: binary SNP overlap table
- `vennCounts.tsv` -> `vennCounts_lncrnasnp.tsv`: overlapped SNP numbers
- `snp_7_core.bed` -> `snp_7_core_lncrnasnp.bed`: SNP `BED` format file

![](fig/euler_snp_26_core_tfbs_snp_74_gtex_enh.png)

```cmd
Rscript venn.r data/seedSNP_5245_bm.bed data/snp_2676_gtex.bed data/snp_1253_encode_dist.bed data/snp_1688_roadmap_dist.bed data/snp_301_regulome2b.bed data/snp_261_lncrnasnp.bed
```

> [1] "seedSNP_5245_bm"
> [1] "snp_2676_gtex"
> [1] "snp_1253_encode_dist"
> [1] "snp_1688_roadmap_dist"
> [1] "snp_301_regulome2b"
> [1] "snp_261_lncrnasnp"
>
> Message: Can't plot Venn diagram for more than 5 sets.
> File write: data/venn.tsv
> File write: data/vennCounts.tsv
>
> Job done for 0.3 sec

- `venn.tsv` -> `summary.tsv` : Summary file for this analysis.
- `vennCounts.tsv` -> `summaryCount.tsv` : Summary SNP numbers.


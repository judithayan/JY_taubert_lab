# GSEA tutorial

Taubert lab's GSEA workflow.

Required: DEG data in csv / xlsx format





## Preparing gmt files

GMT are tab-delimited files, with group name In first/ second columns, and gene list in 3rd-nth column.



### Download databases

#### C. elegans

1. Wormenrichr: https://amp.pharm.mssm.edu/WormEnrichr/#stats
2. g:profiler



### Preparing gmt file for GSEA

Check if the first 2 columns are empty. Gene list should start from column 3 or beyond.

If the format is correct, simply change extension to .gmt, and use with GSEA.



#### Preparing gmt with awk in command line

If the txt gene list starts at 2nd column, need to manually add one column each line, containing either the index, or the group name in duplicate. This can be done by 

1. Open linux console and navigate to folder using `cd` command 
   - if using Linux subsystem for Windows, navigate out of the root folder using `/mnt/Users/User/...`
2. To duplicate first line:
   ` awk -F'\t' '{print $1 "\t" $0}' InterPro_Domains_2019.txt > ceInterpro.gmt`
    (the default separator for `awk` is space, manually change to tab by `-F’\t’`. The rest of the script is what you want to output per line, `$1` = 1st element, `$0` = whole line)
3. To set first line to index:
   ``awk '{print int(NR) "\t" $0}' InterPro_Domains_2019.txt > ceInterpro.gmt``

 

#### Other operations to edit gmt files with awk in command line

##### Concatenating strings to a column.

For instance I want to annotate all the term names with database name ("IP"), so I can concatenate multiple databases without mixing them up.

```
awk -F'\t' -vOFS='\t' '{ $1 = $1 " (IP)"}1' InterPro_Domains_2019.txt> InterPro_Domains_2019_.txt
```

- `-F'\t'` reads the file in tab delimited
- `-vOFS='\t'` outputs to tab delimited
- `'{ $1 = $1 " (IP)"}1'` in the brackets are your command. `$1` = first element, `" (IP)"` is the string to add, and final `1` means print the new line out even though we did something to it.



##### Find the maximum/ minimum gene set length in a db. 

```
cat KEGG_2019.gmt | awk -F'\t' '{ print NF-2 }' | sort -n | tail -1
```

- `{ print NF-2 }` `NF` stands for number of fields (in line).  Assuming gene set starts from 3rd column, this returns max gene set size.

To find minimum gene set size:

```
cat KEGG_2019.gmt | awk -F'\t' '{ print NF-2 }' | sort -n | head -1
```

(note: these are estimates because there could be blank fields in the list)

 

## Preparing rnk files

How to generate a RNK file from the edgeR output (in csv format).

RNK files are tab-delimited like gmt files. An RNK file contains 2 columns: “GeneName”  and “Rank”.

- GeneName is simply the gene symbols column you find in the DEG csv.
- Rank can be calculated very easily in excel. The formula is `=-SIGN(logFC)*LOG10(Pvalue)`. 

##### Generating rnk files with excel

- Paste the two columns in a csv file and export a tab-delimited txt using excel (important: don’t have more than 2 columns). 
- Then change txt extension to .rnk.

##### Generating rnk files witih vim (command line)

- `cd` to a folder 
- `vim filename.rnk` 
- select columns from excel, and paste columns in the interface (may take a while to load up)
- hit `esc`, then hit `shift` to bring up the command line
- type `wq` to save file. (`wq` = write and quit, `q!`= don’t save and quit) 



## Using GSEA

Download GSEA from https://www.gsea-msigdb.org/gsea/index.jsp (requires free registration and installation)

You can read this paper for detailed instructions: https://www.nature.com/articles/s41596-018-0103-9

### Load data

You need 2 types of files:

- rnk (multiple)
- gmt (1 or multiple)

Loading will put them in alphabetical order. If you want them to appear in a specific order, load first batch, click "clear", and then load second batch.

You can always come back to this tab and load more files during analysis.

### GSEA Preranked: parameters

We are using Preranked because rnk files are preranked.

| Parameter                                      | input                                                        |
| ---------------------------------------------- | ------------------------------------------------------------ |
| Gene sets database                             | select your file under local gmt                             |
| n. of permutations                             | 1000 (default)                                               |
| Ranked list                                    | select rnk                                                   |
| Collapse/ remap                                | No_collapse                                                  |
| Analysis name                                  | your_name                                                    |
| Enrichment_statistic                           | weighted (default)                                           |
| Max / min size                                 | Analysis only looks at gene sets in a certain size range. If you want the full set, check the max/ min size before proceeding |
| Plot graphs for the top sets of each phenotype | default = 20, turn this up if want more GSEA plots           |

Run GSEA. (multiple can run at the same time, but slow)

### GSEA results

Report summaries (index.html) are accessible on the bottom left corner.

You can also extract information by directly looking in the output folder:

- `index.html` - summary
- `gsea_report_for_na_neg_seed.html`, `gsea_report_for_na_pos_seed.html` - html table for downregulated and upregulated pathways, containing full statistics. Useful for making summary visualizations.
- `gene_set_sizes.xls` - shows which datasets are excluded, how many genes are found, etc
- xls and html are available for each gene set, detailing gene-level information
- `neg_snapshot.html`, `neg_snapshot.html` - shows enrichment plots for top pathways
- `pvalues_vs_nes_plot.png` - the p-value vs NES plot
- `global_es_histogram.png` - histogram showing distribution of ES
- `ranked_gene_list_na_pos_versus_na_neg_seed.xls` - the original ranked list
- the `edb` folder stores the rnk and gmt used.


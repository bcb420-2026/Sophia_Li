# Exercises

These are intended to be done **after** completing the worked examples.

## Exercise 1 — https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE119732

Using **GSE119732**, confirm whether the ID column contains Ensembl IDs with version suffixes.

1. Extract the first 20 IDs.
2. Count how many contain a `.`.
3. Create a new column with versions stripped.
4. Map the identifiers to HGNC symbols.

- query for the dataset and read its raw table

```r
source("./fetch_geo_supp.R")
fetch_geo_supp(gse = "GSE119732")

path <- file.path("data", "GSE119732")
files <- list.files(path, pattern = "\\.txt.gz$|\\.tsv.gz$|\\.csv.gz$",
                    full.names = TRUE, recursive = TRUE)
x <- readr::read_tsv(files[1], show_col_types = FALSE)
```

- extract the first 20 IDs and count how many had versions

```r
ids <- x[[1]][1:20] |> as.character()
sum(grepl("\\.", unlist(ids)))
```

- create a stripped column in the original object

```r
strip_ensembl_version <- function(x) sub("\\..*$", "", x)
x$stripped_gene_id <- strip_ensembl_version(x$gene_id)
```

- map the stripped ensembl identifiers to HGNC symbols
- fuction automatically batches the query

```r
ensembl <- useMart("ensembl")
datasets <- listDatasets(ensembl)
human <- datasets[grep(datasets$dataset, pattern = "hsapiens"), ]
ensembl <- useDataset("hsapiens_gene_ensembl", mart = ensembl)

map <- getBM(
  attributes = c("ensembl_gene_id", "hgnc_symbol"),
  filters = "ensembl_gene_id",
  values = unlist(x$stripped_gene_id),
  mart = ensembl
)
```

- merge the mapping back into the original data to view
- encountered unexpected many-to-many relationship between gene ID and HGNC symbol
- not all IDs were able to be mapped to an HGNC symbol

```r
x <- x %>%
    mutate(ensembl_gene_id = stripped_gene_id) %>%
    left_join(map, by = "ensembl_gene_id") %>%
    dplyr::select(gene_id, hgnc_symbol, everything())
```

## Exercise 2 — https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE122380

Using **GSE122380**, confirm whether the ID column contains Ensembl IDs with version suffixes.

1. Extract the first 20 IDs.
2. Create a new column with versions stripped.
3. Map the identifiers to HGNC symbols.
4. What is different about this file? 

- query for the dataset and read its raw table as a table, rather than a `.txt`
- downloaded the dataset, not locally cached

```r
source("./fetch_geo_supp.R")
fetch_geo_supp(gse = "GSE122380")

path <- file.path("data", "GSE122380")
files <- list.files(path, pattern = "\\.txt.gz$|\\.tsv.gz$|\\.csv.gz$",
                    full.names = TRUE, recursive = TRUE)
x <- readr::read_table(files[1], show_col_types = FALSE)
```

- extract the first rows, effectively subsetting the file to 20 rows
- worked with this instead to optimize the example
- added a new column for stripped IDs

```r
x <- x[1:20, ]
strip_ensembl_version <- function(x) sub("\\..*$", "", x)
x$ensembl_gene_id <- strip_ensembl_version(x$Gene_id)
```

- upon manual inspection, the gene IDs do not have ensembl versions; stripped is same
- mapped the identifiers to HGNCs

```r
ensembl <- useMart("ensembl")
datasets <- listDatasets(ensembl)
human <- datasets[grep(datasets$dataset, pattern = "hsapiens"), ]
ensembl <- useDataset("hsapiens_gene_ensembl", mart = ensembl)

map <- getBM(
  attributes = c("ensembl_gene_id", "hgnc_symbol"),
  filters = "ensembl_gene_id",
  values = unlist(x$ensembl_gene_id),
  mart = ensembl
)
```

- merged back with the original data
- already added the stripped gene IDs as the `ensembl_gene_id` column

```r
x <- x %>%
    left_join(map, by = "ensembl_gene_id") %>%
    dplyr::select(Gene_id, hgnc_symbol, everything())
```

## Exercise 3 - 
 
Can you use the worked example to process the above two GEO records?  How?

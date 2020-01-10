
<!-- README.md is generated from README.Rmd. Please edit that file -->

# UMI4Cats <img src="man/figures/logo.png" width="121px" height="140px" align="right" style="padding-left:10px;background-color:white;" />

<!-- badges: start -->

[![](https://img.shields.io/badge/lifecycle-maturing-blue.svg)](https://www.tidyverse.org/lifecycle/#maturing)
<!-- badges: end -->

The goal of UMI4Cats is to provide and easy-to-use package to analyze
UMI-4C contact data.

## Installation

    devtools::install_gitlab("pasquali-lab/umi4cats")

Now you can load the package using `library(UMI4Cats)`.

## Basic usage

``` r
library(UMI4Cats)
```

``` r
## 1) Generate Digested genome ----------------------------
# The selected RE in this case is DpnII (|GATC), so the cs5p is "" and cs3p is GATC
hg19_dpnii <- digestGenome(cut_pos = 0,
                           res_enz = "GATC",
                           name_RE = "DpnII",
                           refgen = BSgenome.Hsapiens.UCSC.hg19::BSgenome.Hsapiens.UCSC.hg19, 
                           out_path = "digested_genome/")

## 2) Process UMI-4C fastq files --------------------------
raw_dir <- system.file(file.path("extdata", "SOCS1", "fastq"), 
                       package="UMI4Cats")

contactsUMI4C(fastq_dir = raw_dir,
              wk_dir = "SOCS1",
              bait_seq = "CCCAAATCGCCCAGACCAG",
              bait_pad = "GCGCG",
              res_enz = "GATC",
              cut_pos = 0,
              digested_genome = system.file(file.path("extdata", 
                                                      "digested_genomes",
                                                      "BSgenome.Hsapiens.UCSC.hg19_DpnII"),
                                            package="UMI4Cats"),
              ref_gen = paste0(system.file(file.path("extdata", "ref_genome"),
                                    package="UMI4Cats"), "/ucsc.hg19.chr16.fa"),
              threads = 5)
```

``` r
## 3) Get filtering and alignment stats -------------------
statsUMI4C(wk_dir = system.file("extdata", "SOCS1",
                               package="UMI4Cats"))
```

<img src="man/figures/README-unnamed-chunk-5-1.png" width="100%" />

``` r

## 4) Analyze UMI-4C results ------------------------------
# Load sample processed file paths
files <- list.files(system.file("extdata", "SOCS1", "count", 
                                package="UMI4Cats"),
                    pattern="*_counts.tsv",
                    full.names=TRUE)

# Create colData including all relevant information
colData <- data.frame(sampleID = gsub("_counts.tsv", "", basename(files)),
                      file = files,
                      stringsAsFactors=F)

library(tidyr)
colData <- colData %>% 
  separate(sampleID, 
           into=c("condition", "replicate", "viewpoint"),
           remove=FALSE)

# Load UMI-4C data and generate UMI4C object
umi <- makeUMI4C(colData=colData,
                 viewpoint_name="SOCS1")

## 5) Perform differential test ---------------------------
umi <- fisherUMI4C(umi,
                   filter_low = 20)

## 6) Plot results ----------------------------------------
plotUMI4C(umi, 
          ylim=c(0,10),
          xlim=c(11e6, 11.5e6))
```

<img src="man/figures/README-unnamed-chunk-5-2.png" width="100%" />

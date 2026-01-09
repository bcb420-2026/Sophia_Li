FROM risserlin/bcb420-base-image:winter2026-arm64

# Install BiocManager if not already available
RUN R -e "if (!requireNamespace('BiocManager', quietly = TRUE)) \
          install.packages('BiocManager', repos = 'https://cloud.r-project.org')"

# Install Bioconductor packages
RUN R -e "BiocManager::install('DESeq2', ask = FALSE, update = FALSE)"
RUN R -e "BiocManager::install('enrichplot', ask = FALSE, update = FALSE)"

# Install CRAN packages
RUN R -e "install.packages('pheatmap', repos = 'https://cloud.r-project.org')"  

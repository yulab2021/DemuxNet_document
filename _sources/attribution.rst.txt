Attribution & Driver Identification
===================================

.. image:: _static/DemuxNet_Gene_Countribution_wf.png
   :alt: Attribution workflow
   :align: center
   :width: 700px

This framework uses DemuxNet and Integrated Gradients (IG) to quantify gene contributions.
It introduces two metrics:

- **WIS** (Weighted Importance Score)
- **DCS** (Directional Consistency Score)

Attribution Data Preparation
----------------------------

Prepare a raw count matrix with labels. Replace sample info with generic CMO labels
and prefix the barcodes.

.. code-block:: R

   library(dplyr)
   library(Seurat)
   library(stringr)

   sc <- readRDS('path/to/your/sc.rds')

   count_matrix <- GetAssayData(sc, assay = "RNA", slot = "counts")
   meta_data <- sc@meta.data
   sc1 <- CreateSeuratObject(counts = count_matrix, meta.data = meta_data)

   sc1@meta.data$Barcode <- rownames(sc1@meta.data)
   sc1@meta.data <- sc1@meta.data %>%
     mutate(
       new_Barcode = case_when(
         str_detect(Tissue, "HT") ~ paste0("CMO301_", str_replace(Barcode, ".*_", "")),
         str_detect(Tissue, "NT") ~ paste0("CMO302_", str_replace(Barcode, ".*_", "")),
         TRUE ~ str_replace(Barcode, ".*_", "")
       )
     )

   sc1 <- RenameCells(sc1, new.names = sc1@meta.data$new_Barcode)

   counts_matrix <- LayerData(object = sc1, assay = "RNA", layer = 'counts')
   saveRDS(counts_matrix, file = 'sc_counts.rds')

Run Attribution Analysis
------------------------

.. code-block:: bash

   python demuxnet/ig.py \
     -i test.rds \
     --features 2000 \
     --feature-mode hvg \
     --gene-list Custom.csv \
     --gene-mode all \
     --ig-output ig/ig_summary.csv \
     --randomize-runs 3 \
     --label-map-file label_map.csv \
     --prediction-output-prefix preds/run \
     --accuracy-output ig/accuracy.csv

Attribution Parameters
----------------------

- ``-i``: Input gene expression matrix (RDS).
- ``--features``: Number of top features (default: 6000).
- ``--feature-mode``: ``nonzero`` (default) or ``hvg``.
- ``--gene-list``: Optional CSV/TXT file with additional genes.
- ``--gene-mode``: ``only`` or ``all`` (union).
- ``--ig-output``: Integrated Gradients summary output.
- ``--randomize-runs``: Number of independent runs (0 means single run).
- ``--label-map-file``: CSV mapping original labels to replacement labels.
- ``--prediction-output-prefix``: Prefix for prediction CSVs.
- ``--accuracy-output``: Accuracy summary output.

Attribution Output Structure
----------------------------

.. code-block:: text

   .
   ├── accuracy.csv
   ├── ig_summary_run1_pred.csv
   ├── ig_summary_run1_true.csv
   ├── ig_summary_run2_pred.csv
   ├── ig_summary_run2_true.csv
   ...
   └── preds
       ├── run_run1_prediction.csv
       ├── run_run2_prediction.csv
       └── run_run3_prediction.csv

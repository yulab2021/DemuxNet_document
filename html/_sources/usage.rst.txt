Usage
=====

DemuxNet supports two primary workflows:

1. **Standard Demultiplexing**: Predict missing CMO labels for cells.
2. **Attribution & Driver Identification**: Identify key gene drivers via Integrated Gradients (IG).

Standard Demultiplexing
-----------------------

DemuxNet automatically detects the ``CMO`` keyword within barcode strings for training.
For barcodes missing this information, the model predicts and fills the CMO class.

.. code-block:: bash

   demuxnet -i gene_expression_matrix.rds -model DNN -feature 6000 -out prediction.csv

Parameters
~~~~~~~~~~

- ``-i``: Input sparse expression matrix (RDS). Rows are genes; columns are cells.
- ``-model``: Machine learning model (currently supports ``DNN``).
- ``-feature``: Top features (genes) to select based on non-zero counts.
- ``-out``: Output CSV with two columns: ``Barcode`` and ``Predicted CMO Label``.

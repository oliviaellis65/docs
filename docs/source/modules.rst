Modules
===========

This page describes each nextflow module, including overall purpose, inputs, and outputs. 

Each module is supported by a python or R script which is *run* by a nextflow script. The source code to both files are linked under each module header.

.. raw:: html
   <p>Inputs highlighted in <span style="background-color: #FFCC00; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;">yellow</span><span style="display:inline;"> are required-- others are optional.</span></p>

.. raw:: html
   Outputs highlighted in <span style="background-color: green; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;">green</span><span style="display:inline;"> are used in subsequent modules, or can be supplied by the user if the current module was bypassed.

Jump to:
- :ref:`QC<qc>` 
- :ref:`Process<process>`
- :ref:`Annotate<celltypist>`
- :ref:`Convert<convert>`



.. _qc:

qc_scanpy.nf
--------------

.. note:
   Potentially separate the prep/combination stage and the QC metric generation stage as different processes, same workflow.


Concatenates all samples from the provided samplesheet into a single .h5ad file and generates sample-level QC metrics. If CITE-Seq data is present, GEX and CSP assays are separated. Extra columns from the samplesheet are added to object metadata.

Inputs:
^^^^^^^^^^^
.. raw:: html

   <ul><li><span style="background-color: #FFCC00; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;"> samplesheet.csv:</span><span style="display:inline;">  If starting from cellranger outputs, specify the samples in the following format. "sample" is the sample name, and "dir" is the cellranger output folder containing a .h5ad file, normally in this form: ".../per_sample_outs/sampleA/outs". Additional columns will be added to sample metadata.</span></li></ul>

::

   sample,dir,...
   Donor1_Control,~/myproject/7.2/per_sample_outs/Donor1_Control/outs,
   Donor1_Stim,~/myproject/7.2/per_sample_outs/Donor1_Stim/outs
   Donor2_Control,~/myproject/7.2/per_sample_outs/Donor2_Control/outs


Outputs:
^^^^^^^^^^^^

- **all_samples_gex.h5ad:** Gene expression H5AD file, combined across all samples.
   
   .. raw:: html
         <details>
         <summary>Object metadata</summary>
          all_samples_gex.h5ad contains metadata for QC metrics, including:
            'nFeature_RNA', 'nCount_RNA', 'percent_mt', 'percent_ribo', 'percent_rbc', 'log1p_n_genes_by_counts', 'log1p_total_counts', 'pct_counts_in_top_50_genes', 'pct_counts_in_top_100_genes', 'pct_counts_in_top_200_genes', 'pct_counts_in_top_500_genes', 'total_counts_mt', 'log1p_total_counts_mt',  'total_counts_ribo', 'log1p_total_counts_ribo',  'total_counts_hb', 'log1p_total_counts_hb'
         </details>

- **all_samples_csp.h5ad:** Combined CSP H5AD file if CSP data is present.
- **QC_metrics.xlsx:** Provides 5%, 10%, 90%, and 95% values for 'nFeature_RNA', 'nCount_RNA', 'percent_mt', 'percent_ribo' *across all samples combined*.
- **QC_plot.png:** For each sample, shows the distributions of 'nFeature_RNA', 'nCount_RNA', 'percent_mt', 'percent_ribo', and the number of cells.


.. _process:

process_scanpy.nf
------------------

.. note:
   Potentially separate the filtering

Filters, integrates, and clusters data using scanpy. Parameters are outlined below and set in the nextflow.config file. 

Filtering metrics are listed below and include nFeature and nCount minima and maxima, as well as maximum values for mitochondrial and ribosomal percentage.

Integration may be performed using either Harmony or ScVI. The default method is harmony.

Clustering is performed using the batch-corrected matrix. 

Inputs:
^^^^^^^

.. raw:: html

   <ul><li><span style="background-color: #FFCC00; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;"> all_samples:</span><span style="display:inline;">  file path to an .h5ad object with gene expression data combined for all samples. This can be an output from qc_scanpy.nf, or a user-supplied object (see requirements below).</span></li></ul>

.. raw:: html

   <p><details>
   <summary>Click to expand</summary>
   Object must contain the following metadata columns: 'sample_id', 'nFeature_RNA', 'nCount_RNA', 'percent_mt', 'percent_ribo'.
   </details></p>



.. raw:: html

   <ul><li><span style="color:black;font-weight:bold;">workers</span><span style="display:inline;">: number of workers to use for integration. Default is the number of available workers - 1.</span></li></ul>


.. raw:: html

   <ul><li><span style="background-color: #FFCC00; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;">qc</span><span style="display:inline;"> <i>(all parameters required)</i></span></li></ul>

*

   +-------------------+--------------------------------------------------+----------+----------------+
   | Parameter         | Description                                      | Default  | Type           |
   +===================+==================================================+==========+================+
   | min_nFeature      | Minimum number of unique genes in a cell         | 200      | ``integer``    |
   +-------------------+--------------------------------------------------+----------+----------------+
   | max_nFeature      | Maximum number of unique genes in a cell         | 2000     | ``integer``    |
   +-------------------+--------------------------------------------------+----------+----------------+
   | min_nCount        | Minimum number of total reads in a cell          | 2000     | ``integer``    |
   +-------------------+--------------------------------------------------+----------+----------------+
   | max_nCount        | Maximum number of total reads in a cell          | 10000    | ``integer``    |
   +-------------------+--------------------------------------------------+----------+----------------+
   | percent_mt        | Maximum % of cell reads from mitochondrial genes | 10       | ``float``      |
   +-------------------+--------------------------------------------------+----------+----------------+
   | percent_ribo      | Maximum % of cell reads from ribosomal genes     | 30       | ``float``      |
   +-------------------+--------------------------------------------------+----------+----------------+


- **batch**: *(optional)*
   +------------+------------------------------------------------------------------+--------------+-----------+
   | Parameter  | Description                                                      | Default      | Type      |
   +============+==================================================================+==============+===========+
   | batch      | The metadata column to use as a batch variable for integration   | "sample_id"  | string    |
   +------------+------------------------------------------------------------------+--------------+-----------+
   | integrate  | Method used for integration-- either "harmony" or "scvi"         | "harmony"    | string    |
   +------------+------------------------------------------------------------------+--------------+-----------+
   | var_genes  | Number of variable genes used for batch correction               | 2000         | integer   |
   +------------+------------------------------------------------------------------+--------------+-----------+


- **umap** *(optional)*
   +-------------+------------------------------------------------------------------+-----------+-----------+
   | Parameter   | Description                                                      | Default   | Type      |
   +=============+==================================================================+===========+===========+
   | dimensions  | Number of principle components to use for clustering (1-50)      | 30        | string    |
   +-------------+------------------------------------------------------------------+-----------+-----------+
   | resolution  | Clustering resolution (0.1-1.5)                                  | 0.3       | float     |
   +-------------+------------------------------------------------------------------+-----------+-----------+
   
   

Outputs
^^^^^^^^^


.. _celltypist:

celltypist_annotate.nf
-------------------------

Annotates cells using CellTypist.

Inputs:
^^^^^^^^^^
.. raw:: html

   <ul><li><span style="background-color: #FFCC00; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;">filtered</span><span style="display:inline;">: file path to an .h5ad object with gene expression data combined for all samples. This can be an output from process_scanpy.nf, or a user-supplied object (see requirements below).</span></li></ul>

.. raw:: html

   <p><details>
   <summary>Requirements</summary>
   Object must contain the following metadata columns: 'sample_id', 'nFeature_RNA', 'nCount_RNA', 'percent_mt', 'percent_ribo', 'leiden'.
   </details></p>


Outputs:
^^^^^^^^^^^
- annotated_gex.h5ad: Annotated gene expression object. CellTypist labels are in 'cell.type'.
- cluster_markers.xlsx: Top markers from each cluster, as defined by the 'leiden' metadata column. Markers are calculated only by cluster, and are agnostic to CellTypist label.
- celltypist_markers.xlsx: Markers from the cluster that were used to assign the CellTypist label.


.. _convert:
scanpy_to_seurat.nf
-----------------------

**may need to reduce number of cells to fit inside a seurat object**. Converts Scanpy objects to Seurat objects.

Inputs:
^^^^^^^^^^

.. raw:: html

   <ul><li><span style="background-color: #FFCC00; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;">gex:</span><span style="display:inline;"> Gene expression object</span></li></ul>

- **csp:** CSP object.



Outputs:
^^^^^^^^^^^
- annotated.rds: Annotated Seurat object.

Modules
===========

.. note::
   For each module, check 
      - Inputs error checking
      - Outputs are correct
      
      

This page describes each nextflow module, including overall purpose, inputs, and outputs. 

Each module is supported by a 'core' python or R script which is *run* by a nextflow script. The source code to both files are linked under each module header.

.. raw:: html

   <p>Inputs highlighted in <span style="background-color: #FFCC00; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;">yellow</span><span style="display:inline;"> are required, others are optional.</span></p>


.. raw:: html

   Outputs highlighted in <span style="background-color: green; color: white; font-weight:bold; padding: 2px 6px; border-radius: 4px;">green</span><span style="display:inline;"> are used in subsequent modules, or can be supplied by the user if the current module was bypassed.

.. block end 


Jump to:

- :ref:`QC<qc>` 
- :ref:`Process<process>`
- :ref:`Annotate<celltypist>`
- :ref:`Convert<convert>`



.. _qc:

Prep and Quality Control
--------------

.. note:
   Potentially separate the prep/combination stage and the QC metric generation stage as different processes, same workflow.


Concatenates all samples from the provided samplesheet into a single .h5ad file and generates sample-level QC metrics. If CITE-Seq data is present, GEX and CSP assays are separated. Extra columns from the samplesheet are added to object metadata.



.. raw:: html

   <p><span style="background-color: pink; font-size: 18px; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;">SCRIPTS</span> <a href="https://github.com/EliLillyCo/nf-ellis-scrnaseq/blob/main/modules/qc_scanpy.nf"> qc_scanpy.nf, </a> <a href="https://github.com/EliLillyCo/nf-ellis-scrnaseq/blob/main/bin/qc_scanpy.py">qc_scanpy.py</a></p>

.. raw block end 


Inputs
^^^^^^^^^^^
.. raw:: html

   <ul><li><span style="background-color: #FFCC00; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;"> samplesheet.csv</span><span style="display:inline;">:  If starting from cellranger outputs, specify the samples in the following format. "sample" is the sample name, and "dir" is the cellranger output folder containing a .h5ad file, normally in this form: ".../per_sample_outs/sampleA/outs". Additional columns will be added to sample metadata.</span></li></ul>

::

   sample,dir,...
   Donor1_Control,~/myproject/7.2/per_sample_outs/Donor1_Control/outs,
   Donor1_Stim,~/myproject/7.2/per_sample_outs/Donor1_Stim/outs
   Donor2_Control,~/myproject/7.2/per_sample_outs/Donor2_Control/outs



Outputs
^^^^^^^^^^^^

.. raw:: html

   <ul><li><span style="background-color: green; color: white; font-weight:bold; padding: 2px 6px; border-radius: 4px;">all_samples_gex.h5ad</span><span style="display:inline; color: black;">: Gene expression H5AD file, combined across all samples.</span></li></ul>
   <ul><li><span style="background-color: green; color: white; font-weight:bold; padding: 2px 6px; border-radius: 4px;">all_samples_csp.h5ad</span><span style="display:inline; color: black;">: Combined CSP H5AD file if CSP data is present.</span></li></ul>


.. raw:: html

   <details style="margin-left: 40px;">
      <summary>Requirements no span</summary>
       all_samples_gex.h5ad contains metadata for QC metrics, including: 'nFeature_RNA', 'nCount_RNA', 
   </details>

.. raw:: html

   <details style="margin-left: 40px;">
      <summary><span style="font-size: 12px; margin-left: 40px;">Requirements w span</span></summary>
       all_samples_gex.h5ad contains metadata for QC metrics, including: 'nFeature_RNA'
   </details>

.. fake comment

- **QC_metrics.xlsx:** Provides 5%, 10%, 90%, and 95% values for 'nFeature_RNA', 'nCount_RNA', 'percent_mt', 'percent_ribo' *across all samples combined*.
- **QC_plot.png:** For each sample, shows the distributions of 'nFeature_RNA', 'nCount_RNA', 'percent_mt', 'percent_ribo', and the number of cells.

.. 'percent_mt', 'percent_ribo', 'percent_rbc', 'log1p_n_genes_by_counts', 'log1p_total_counts', 'pct_counts_in_top_50_genes', 'pct_counts_in_top_100_genes', 'pct_counts_in_top_200_genes', 'pct_counts_in_top_500_genes', 'total_counts_mt', 'log1p_total_counts_mt',  'total_counts_ribo', 'log1p_total_counts_ribo',  'total_counts_hb', 'log1p_total_counts_hb'

.. _process:

PROCESS
------------------

.. note::
   Potentially separate the filtering
   Fix config params to fail

Filters, integrates, and clusters data using scanpy. The parameters for each component step are outlined below, and set in the **nextflow.config** file. While parameters are only *required* for the filtering step, I encourage inspecting the batch correction and umap parameters as well.

.. note::

   By default, qc parameters are left null, which **causes the pipeline to fail after the QC module is completed**. It sounds scary, but this is the desired behavior! 
   Failing after QC allows the user to inspect quality metrics and determine appropriate thresholds at runtime. Once parameters are specified, the pipeline continues where it left off with cached temporary objects. 

Quality metrics for filtering cells include nFeature and nCount minima and maxima, as well as maximum values for mitochondrial and ribosomal percentage.

Integration may be performed using either Harmony or ScVI. The default method is Harmony.

Clustering is performed using the batch-corrected matrix from either ScVI or Harmony. 

.. raw:: html

   <p><span style="background-color: pink; font-size: 18px; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;">SCRIPTS</span> <a href="https://github.com/EliLillyCo/nf-ellis-scrnaseq/blob/main/modules/process_scanpy.nf"> process_scanpy.nf, </a> <a href="https://github.com/EliLillyCo/nf-ellis-scrnaseq/blob/main/bin/process_scanpy.py">process_scanpy.py</a></p>
.. raw block end 


Inputs
^^^^^^^^^

.. raw:: html

   <ul><li><span style="background-color: #FFCC00; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;"> all_samples</span><span style="display:inline;">:  file path to an .h5ad object with gene expression data combined for all samples. This can be an output from qc_scanpy.nf, or a user-supplied object (see requirements below).</span></li></ul>

.. raw:: html
      <p><details>
      <summary><span style="font-size: 12px; margin-left: 40px;">Requirements</span></summary>
      Object must contain the following metadata columns: 'sample_id', 'nFeature_RNA', 'nCount_RNA', 'percent_mt', 'percent_ribo'.
      </details></p>



.. raw:: html

   <ul><li><span style="color:black;font-weight:bold;">workers</span><span style="display:inline;">: number of workers to use for integration. Default is the number of available workers - 1.</span></li></ul>

.. raw:: html

   <ul><li><span style="background-color: #FFCC00; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;">qc <i>(all parameters required)</i></span></li></ul>


.. doin stuff


   +-------------------+--------------------------------------------------+----------+----------------+
   | Parameterz        | Description                                      | Default  | Type           |
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


- **batch**  *(optional)*
   +------------+------------------------------------------------------------------+--------------+-----------+
   | Parameter  | Description                                                      | Default      | Type      |
   +============+==================================================================+==============+===========+
   | batch      | The metadata column to use as a batch variable for integration   | "sample_id"  | string    |
   +------------+------------------------------------------------------------------+--------------+-----------+
   | integrate  | Method used for integration-- either "harmony" or "scvi"         | "harmony"    | string    |
   +------------+------------------------------------------------------------------+--------------+-----------+
   | var_genes  | Number of variable genes used for batch correction               | 2000         | integer   |
   +------------+------------------------------------------------------------------+--------------+-----------+


- **umap**   *(optional)*
   +-------------+------------------------------------------------------------------+-----------+-----------+
   | Parameter   | Description                                                      | Default   | Type      |
   +=============+==================================================================+===========+===========+
   | dimensions  | Number of principle components to use for clustering (1-50)      | 30        | string    |
   +-------------+------------------------------------------------------------------+-----------+-----------+
   | resolution  | Clustering resolution (0.1-1.5)                                  | 0.3       | float     |
   +-------------+------------------------------------------------------------------+-----------+-----------+
   
   

Outputs
^^^^^^^^^

.. raw:: html
   <ul><li><span style="background-color: green; color: white; font-weight:bold; padding: 2px 6px; border-radius: 4px;">filtered_gex.h5ad</span><span style="display:inline;": Filtered, batch corrected, clustered GEX object. Used in ANNOTATE and CONVERT. </span></li></ul>
   <ul><li><span style="background-color: green; color: white; font-weight:bold; padding: 2px 6px; border-radius: 4px;">filtered_csp.h5ad</span><span style="display:inline;": CSP object filtered to the same cells as the GEX object. Not clustered or batch corrected. Used in ANNOTATE and CONVERT. </span></li></ul>

- **umap_plot.png:**
- **process.log:** 



.. _celltypist:

ANNOTATE
-------------------------
.. note:: 
   Need to check if behavior is correct; do we really need the QC info?

Annotates cells using `CellTypist<https://www.celltypist.org/>`_.

.. raw:: html

   <p><span style="background-color: pink; font-size: 18px; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;">SCRIPTS</span> <a href="https://github.com/EliLillyCo/nf-ellis-scrnaseq/blob/main/modules/celltypist_annotate.nf"> celltypist_annotate.nf, </a> <a href="https://github.com/EliLillyCo/nf-ellis-scrnaseq/blob/main/bin/celltypist_annotate.py">celltypist_annotate.py </a></p>
.. raw block end 


Inputs
^^^^^^^^^^
.. raw:: html

   <ul><li><span style="background-color: #FFCC00; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;">filtered</span><span style="display:inline;">: file path to an .h5ad object with gene expression data combined for all samples. This can be an output from process_scanpy.nf, or a user-supplied object (see requirements below).</span></li></ul>

.. raw:: html

      <p><details>
      <summary><span style="font-size: 12px; margin-left: 40px;">Requirements</span></summary>
      Object must contain the following metadata columns: 'sample_id', 'nFeature_RNA', 'nCount_RNA', 'percent_mt', 'percent_ribo', 'leiden'.
      </details></p>


Outputs
^^^^^^^^^^^

.. raw:: html
   <ul><li><span style="background-color: green; color: white; font-weight:bold; padding: 2px 6px; border-radius: 4px;">annotated_gex.h5ad</span><span style="display:inline;": Annotated gene expression object. CellTypist labels are stored in the 'cell.type' metadata variable. Used in CONVERT. </span></li></ul>

- **cluster_markers.xlsx:** Top markers from each cluster, as defined by the 'leiden' metadata column. Markers are calculated only by cluster, and are agnostic to CellTypist label.
- **celltypist_markers.xlsx:** Markers from the cluster that were used to assign the CellTypist label.



.. _convert:
CONVERT
-----------------------

.. note:: 
   Add support for raw counts, CSP


Converts Scanpy objects to Seurat objects using the `cellgeni/schard tool<https://www.bing.com/search?q=cellgeni%20schard%20&qs=n&form=QBRE&sp=-1&lq=0&pq=cellgenischard%20&sc=0-15&sk=&cvid=E5061AD0B4A74611894027B4AA6BF984>`_. Retains dimensionality reductions and metadata, and re-combines CSP and GEX assays (if applicable). With current versions, these Seurat objects can only handle matrices with 2^31 elements or less, i.e. around 100k cells. If the input .h5ad object "gex" exceeds 100k cells, it will **automatically** be subsetted to 100k cells using scanpy's `subsample function<https://scanpy.readthedocs.io/en/stable/generated/scanpy.pp.subsample.html>`_.

.. raw:: html

   <p><span style="background-color: pink; font-size: 18px; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;">SCRIPTS</span> <a href="https://github.com/EliLillyCo/nf-ellis-scrnaseq/blob/main/modules/scanpy_to_seurat.nf"> scanpy_to_seurat.nf, </a> <a href="https://github.com/EliLillyCo/nf-ellis-scrnaseq/blob/main/bin/scanpy_to_seurat.py">scanpy_to_seurat.py</a></p>
.. raw block end 



Inputs
^^^^^^^^^^

.. raw:: html

   <ul><li><span style="background-color: #FFCC00; color: black; font-weight:bold; padding: 2px 6px; border-radius: 4px;">gex</span><span style="display:inline;">:  Gene expression object, either an output from Process, Annotate, or a user-supplied object with appropriate metadata ?? </span></li></ul>
- **csp:** CSP object.



Outputs
^^^^^^^^^^^
- **converted.rds:** Converted Seurat object. GEX data is stored in the "RNA" assay. CSP data, if supplied, is stored in the "CSP" assay. 



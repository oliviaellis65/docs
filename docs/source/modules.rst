Modules
===========

Descriptions of each module, including required inputs and outputs. Black inputs are required and gray inputs are optional.
- [qc_scanpy.nf](#qc_scanpynf)
- [process_scanpy.nf](#process_scanpynf)
- [celltypist_annotate.nf](#celltypist_annotatenf)
- [scanpy_to_seurat.nf](#scanpy_to_seuratnf)

qc_scanpy.nf
--------------

Concatenates all samples from the provided samplesheet into a single .h5ad file and generates sample-level QC metrics. If CITE-Seq data is present, GEX and CSP assays are separated. Extra columns from the samplesheet are added to object metadata.

Inputs:
^^^^^^^^^^^

- **samplesheet.csv:** If starting from cellranger outputs, specify the samples in the following format. "sample" is the sample name, and "dir" is the cellranger output folder containing a .h5ad file, normally in this form: ".../per_sample_outs/sampleA/outs". Additional columns will be added to sample metadata. ::

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


process_scanpy.nf
------------------

Filters, integrates, and clusters data using scanpy. Parameters are outlined below and set in the nextflow.config file. Required parameters are highlighted in yellow. **ADD INFO ABOUT DEFAULT BEHAVIOR**

Inputs:
^^^^^^^

.. raw:: html

   <p><span style="background-color: blue; color: white; font-weight:bold; padding: 2px 6px; border-radius: 4px;">all_samples:</span><span style="display:inline;">file path to an .h5ad object with gene expression data combined for all samples. This can be an output from qc_scanpy.nf, or a user-supplied object (see requirements below).</span>



.. raw:: html

   <details>
   <summary>Click to expand</summary>
   Object must contain the following metadata columns: 'sample_id', 'nFeature_RNA', 'nCount_RNA', 'percent_mt', 'percent_ribo'.
   </details>


.. raw:: html

   <span style="color:gray;"><strong>workers</strong></span>: 

number of workers to use for integration. Default is the number of available workers - 1.

**Filtering (qc)**

+-------------------+--------------------------------------------------+----------+--------------+
| Parameter         | Description                                      | Default  | Type         |
+===================+==================================================+==========+==============+
| min_nFeature      | Minimum number of unique genes in a cell         | 200      | **integer**  |
+-------------------+--------------------------------------------------+----------+--------------+
| max_nFeature      | Maximum number of unique genes in a cell         | 2000     | **integer**  |
+-------------------+--------------------------------------------------+----------+--------------+
| min_nCount        | Minimum number of total reads in a cell          | 2000     | **integer**  |
+-------------------+--------------------------------------------------+----------+--------------+
| max_nCount        | Maximum number of total reads in a cell          | 10000    | **integer**  |
+-------------------+--------------------------------------------------+----------+--------------+
| percent_mt        | Maximum % of cell reads from mitochondrial genes | 10       | **float**    |
+-------------------+--------------------------------------------------+----------+--------------+
| percent_ribo      | Maximum % of cell reads from ribosomal genes     | 30       | **float**    |
+-------------------+--------------------------------------------------+----------+--------------+

Outputs
^^^^^^^


## celltypist_annotate.nf

Purpose: Annotates cells using CellTypist.
Inputs:
filtered: A filtered object.
Outputs:
annotated_gex.h5ad: Annotated gene expression object.
cluster_markers.xlsx: Cluster markers Excel file.
celltypist_markers.xlsx: CellTypist markers Excel file.

## scanpy_to_seurat.nf
**may ned to reduce number of cells to fit inside a seurat object**
Purpose: Converts Scanpy objects to Seurat objects.
Inputs:
gex: Gene expression object.
csp: CSP object.
Outputs:
annotated.rds: Annotated Seurat object.
Each of these files contains a Nextflow process that defines specific steps in the pipeline, taking specific inputs and producing outputs essential for single-cell CITE-Seq data analysis.

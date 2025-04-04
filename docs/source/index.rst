nf-ellis-scrnaseq Documentation
===================================

.. note::

   This project is under active development, and is yet to be released.

This pipeline runs single cell analysis downstream of cellranger. It is compatibe with 10x gene expression or CITE-Seq data. There are 6 central components of the pipeline, with built-in breakpoints for user inspection and quality assurance. Components are highly modular and allow users to enter and exit the pipeline at multiple stages. Built on nextflow, it offers seamless container usage and reproducibility across compute environments. Data processing is done with scanpy to ensure consistent performance for large (>200k cells) datasets. Analysis downstream of clustering is done with R.

The pipeline schematic is pictured below. Each "pipe" represents one nextflow process. A top opening in the pipe indicates a potential starting point. Bottom openings are breakpoints where the user must specify necessary parameters _or_ exit the pipeline. These parameters can either be determined at runtime, or pre-specified so that the pipeline runs seamlessly. Once parameters are supplied, the pipeline continues executing using nextflow's "--resume" feature to cache data. Users can elect to bypass the annotate and analyze processes (gray pipes).


Contents
--------

.. toctree::

   overview
   modules

Overview
==========
The pipeline schematic is pictured below. Each "pipe" represents one nextflow process. 
A top opening in the pipe indicates a potential starting point. 
Bottom openings are breakpoints where the user must specify necessary parameters _or_ exit the pipeline. 
These parameters can either be determined at runtime, or pre-specified so that the pipeline runs seamlessly. Once parameters are supplied, the pipeline continues executing using nextflow's "--resume" feature to cache data. Users can elect to bypass the annotate and analyze processes (gray pipes).

**TODO**

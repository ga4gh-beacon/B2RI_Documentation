# Beacon v2 Reference Implementation

To demonstrate Beacon v2 capabilities and to facilitate the adoption, [ELIXIR](https://elixir-europe.org) organization has been funding the development of the **Beacon v2 Reference Implementation** (B2RI). Developed at the **Centre for Genomic Regulation** ([CRG](https://www.crg.eu)), the B2RI is a free _open source_ **Linux-based** set of tools that allow _lighting up_ a Beacon out-of-the-box. 

The B2RI includes: 

  1. Tools for loading metadata (including phenotypic data) and genomic variants (from a [VCF](https://en.wikipedia.org/wiki/Variant_Call_Format) file) into a database. 
  2. A [MongoDB](https://www.mongodb.com) database.
  3. The Beacon query engine (i.e., REST API).
  4. An example [dataset](https://www.cineca-project.eu/cineca-synthetic-datasets) consisting of synthetic-data (CINECA synthetic cohort EUROPE UK1).

The **B2RI** is conceived as a customizable **local** solution, delivered with a basic configuration. The software is written in ```Python, Perl``` and ```Bash```.

!!! Warning "Acknowledgement"
    This study was funded by ELIXIR, the research infrastructure for life-science data (ELIXIR Beacon Implementation Studies 2019-2021 and 2022-2023).

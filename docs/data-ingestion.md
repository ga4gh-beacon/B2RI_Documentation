# Data ingestion

Data ingestion consists of 3 steps:

1. Transforming **metadata** (including phenotypic data).
2. Transforming **genomic variations** (VCF).
3. Load data into **MongoDB**.

!!! Important
    Here we will give you an overview of the tree steps. For an in depth explanation, please follow this [tutorial](./tutorial-data-beaconization.md).

## 1. Transforming Metadata (Including Phenotypic Data)

![Data Ingestion 1](img/data-ingestion-1.png)

**B2RI** facilitates transforming your data—such as sequencing methodologies, bioinformatics tools, and phenotypic data—into the format defined by the [Beacon v2 Models](http://docs.genomebeacons.org/), using the [bff-validator](https://github.com/mrueda/beacon2-ri-tools/tree/main/utils/bff_validator) utility.

The [Beacon v2 Models](http://docs.genomebeacons.org/) establish the default data structure (schema) for biological data responses. These models are defined using [JSON Schema](https://json-schema.org) and organized hierarchically. The schemas consist of multiple properties or [terms](http://docs.genomebeacons.org/schemas-md/beacon_terms/) (also known as objects).

We have chosen **MongoDB** as a _de facto_ database because it natively supports JSON documents. This allows us to store data directly in the database according to the [Beacon v2 Models](http://docs.genomebeacons.org/) and provide Beacon v2 compliant responses without needing to re-map the data at the API level.

## 2. Transforming Genomic Variations (VCF)

![Data Ingestion 2](img/data-ingestion-2.png)

For genomic data, **B2RI** provides a tool (`beacon`) that takes a [VCF](https://en.wikipedia.org/wiki/Variant_Call_Format) file as input and uses [BCFtools](http://samtools.github.io/bcftools/bcftools.html), [SnpEff](http://pcingola.github.io/SnpEff), and SnpSift to **annotate** the data. Once annotated, the tool transforms the VCF data into the `genomicVariations` entry type defined by the Beacon v2 Models and serializes it into a JSON file.

## 3. Loading Data into MongoDB

![Data Ingestion 3](img/data-ingestion-3.png)

After transformation, the JSON files adhere to what we call the **Beacon Friendly Format** (BFF).

The final step is loading the **BFF** files into a [MongoDB](https://www.mongodb.com) instance. **B2RI** provides a tool (`beacon`) that loads BFF files into MongoDB. Once loaded, these files are referred to as **collections**.

## Included Utilities

In addition to the `beacon` script and `bff-validator`, the data ingestion tools include several [utilities](https://github.com/mrueda/beacon2-ri-tools/tree/main/utils/README.md) to assist with data processing and beyond:

Furthermore, the `beacon2-ri-tools` repository includes the [CINECA synthetic dataset](synthetic-dataset.md).

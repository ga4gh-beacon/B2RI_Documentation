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

In addition to the `beacon` script and `bff-validator`, the data ingestion tools include several utilities to assist with data processing and beyond:

* [bff-api](https://github.com/mrueda/beacon2-ri-tools/tree/main/utils/bff_api)
* [bff-queue](https://github.com/mrueda/beacon2-ri-tools/tree/main/utils/bff_queue)

Furthermore, the `beacon2-ri-tools` repository includes the [CINECA synthetic dataset](synthetic-dataset.md).

## BFF Genomic Variations Browser

!!! Warning "Important"
    BFF Genomic Variations Browser **is not a full UI** for Beacon v2 as it does not allow for cross-queries to other collections (e.g., individuals).

---

### Overview  
The **BFF Genomic Variations Browser** provides a straightforward way to visualize `genomicVariations` documents (stored as a JSON array) through dynamic, HTML-embedded tables.

![BFF Genomic Variations Browser](img/BFF-genomic-variations-browser.png)

The browser is designed to facilitate the analysis of variants with a **HIGH** impact annotation value, offering a convenient tool for targeted exploration.

---

### How to Run

To enable the BFF Genomic Variations Browser, set the following in your parameters file:

```yaml
bff2html: true
```

Once the process is complete, the results will be available in the `<job_id>/vcf/browser` directory.

### Features  

1. **Gene Panel Support**  
   - Variants are displayed in **HTML tabs** organized by gene panels.  
   - **Gene Panels**: Simple text files with a `.lst` extension, containing a single column of gene names.  
   - **Default Directory**: `"$beacon_path/browser/data"`.  
   - **Customization**: You can modify the directory using the `paneldir` parameter in the `config.yaml` file.  
   - **Extendability**: Additional gene panels can be created and added.

2. **Dynamic Tables**  
   - The browser generates searchable and sortable tables directly in HTML.  
   - **Key Features**:
     - Column reordering.  
     - Advanced search with regular expressions (e.g., `rs12(3|4) (tp53|ace2) splice`).  

3. **Filtered Display**
   - Only variants with **HIGH** annotation impact values are included.
   - Variants are filtered and displayed according to the `.lst` files in the `paneldir` folder.

![BFF Genomic Variations Browser - Table View](img/snapshot-BFF-genomic-variations-browser.png)

---

### Loading the HTML  

To view the HTML:  

1. Make sure you are in the results directory (e.g., `<job_id>/browser`) and start Python's built-in HTTP server:  

```bash
python3 -m http.server 8000 --bind 0.0.0.0
```  

2. Open a web browser and navigate to:  

`http://0.0.0.0:8000`

3. Load the desired `job_id.html` file.

---

### Additional Information  

- The generated HTML files are **local** and rely on a JSON file to display data.  
- By default, the browser processes all `.lst` files in the `paneldir` folder for filtering and organizing variants. 

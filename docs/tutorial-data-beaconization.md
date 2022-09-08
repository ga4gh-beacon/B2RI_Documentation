# Tutorial (Data "beaconization")

Let's start by the simplest case. Imagine that you have:

  * **Metadata** (including phenotypic data) in your system, labeled according to your internal nomenclature.
  *  A **VCF** file.

## STEP 1

First of all, we are going to convert your metadata (sequencing methodology, bioinformatics tools, phenotypic data, etc.) to the format of the [Beacon v2 Models](http://docs.genomebeacons.org/schemas-md/analyses_defaultSchema). As **input** we will be using this [XLSX](https://github.com/EGA-archive/beacon2-ri-tools/blob/main/utils/bff_validator/Beacon-v2-Models_template.xlsx) template.

!!! Important "About the XLSX template"
    The XLSX template consists of **seven sheets** that match the [Beacon v2 Models](http://docs.genomebeacons.org/).
    The template has the purpose of facilitating users' transformation of the data (likely in tabular form) to the hierarchical structure that we have in the [Beacon v2 Models](http://docs.genomebeacons.org/schemas-md/analyses_defaultSchema).
    The header nomenclature gives a hint about if the data will be later stored as an `object` (naming contains `.`) or as an `array` (naming contains `_`).
    Each column has its own format (e.g., string, date, [CURIE](https://en.wikipedia.org/wiki/CURIE)). These formats can browsed in the [documentation](http://docs.genomebeacons.org/schemas-md/analyses_defaultSchema). 
    We recommend using the provided XLSX for the [synthetic data](https://github.com/EGA-archive/beacon2-ri-tools/blob/main/CINECA_synthetic_cohort_EUROPE_UK1/Beacon-v2-Models_CINECA_UK1.xlsx) as a reference.

  The first thing that needs to be done is to **map/convert your metadata** so that it follows the syntax of the provided [XLSX](https://github.com/EGA-archive/beacon2-ri-tools/blob/main/utils/bff_validator/Beacon-v2-Models_template.xlsx) file.

![Excel template](img/excel-template.png)


!!! Warning "Note"
    Normally, people don't fill out the sheet (tab) named `genomicVariations` as this info will be taken from the annotated VCF (see STEP 2).

Once you have filled the Excel file then you can proceed to validate it. At this stage, it's normal that you have doubts regarding your mapping to Beacon v2 syntax. Fortunately, B2RI's utility `bff-validator` will help you complete this task. The validator **checks that all values in the XLSX match the specifications present in the Beacon v2 Models default schemas**. In technical terms, it's called _validating data against JSON Schemas_.

    ./utils/bff_validator/bff-validator -i your_xlsx_file.xlsx --out-dir my_bff_dir

When you run it, it's very likely that you'll find have errors/warnings on your data. The script will catch them and explain the cause of the error. Please **address all the issues** at the XLSX. You can run the script as many times you want :-)

![BFF validator](img/bff-validator.png)

!!! Danger "Common errors" 

    **Unicode**

    [Unicode](https://en.wikipedia.org/wiki/UTF-8) characters are allowed as _values_ for the cells. However, if you are copy-pasting from other sources, sometimes "strange" characters are randomly introduced in places where they should not be. If `bff-validator` is giving you errors and you can't figure out how to solve them use the flag `--ignore-validation` and take a look to the _JSON_ files created. Once you spot the error(s), please fix the original Excel file and re-run the validation. See extended information [here](https://github.com/EGA-archive/beacon2-ri-tools/tree/main/utils/bff_validator).
    
    **Specification mismatches**

    [Beacon v2 Models specification](https://github.com/ga4gh-beacon/beacon-v2/tree/main/models) is continously evolving. The changes are not only to _add new and better features_ but also to _fix previous problems_. In this regard, it can happen that `bff-validator` gives some warnings on things that look OK (such as in properties `oneOf`,`anyOf`, etc. coming from Phenopacket specification). If this happens just use the flag `--ignore-validation` once you are ready to create your `.json` files.
    
At some point you won't get any errors. By now the script should have created 6 text files (what we call the **Beacon Friendly Format**). The files are in **JSON** format (JSON arrays) and will be used later (STEP 3).

Congratulations! Now you can go to STEP 2.

## STEP 2

Now that you have processed the metadata, it's time to process the **VCF** file.

!!! Important "About VCF types"
    Currently, the B2RI only handles VCFs coming from **DNAseq** experiments (WES, WGS, gene panels, etc.). The VCFs can be _single_ or _multisample_. <br />
    At the time of writting this, **structural variants** in VCF are not being parsed (there is scout working group currently developing Beacon v2 specifications for structural variants). We hope to implement this feature in future versions. 

The VCF file has to be gzipped (or bgzipped). What we are going to do it's to annotate it (or re-annotate it if your file already has annotations) with **SnpEff and SnpSift** and transform the format so that it becames the 7th BFF file (i.e., `genomicVariationsVcf.json`).

    ./beacon vcf   -n 1    -i input.vcf.gz -p param_file
         |      |      |         |              |
         exe    mode   #cores   <vcf>           parameters file (optional)

Here we are using `beacon` script in mode ***vcf***. This mode is one of the three available [vcf|mongodb|full]. 

The parameters file is needed if you want to change the default values. See all the script options [here](https://github.com/EGA-archive/beacon2-ri-tools).

!!! Important
    **Note about timing**: We made the script _as fast as we possibly could_ with a scripting language. In this regard, the processing time scales linearly with the #variants, but it's also affected by the #samples. For instance, 1M variants with 2,500 samples will take around ~20-25 min.

If something is wrong with the input files, the script will complain and provide possible solutions.

![Beacon to VCF](img/beacon-vcf.png)

Once completed, you will end up with a dir like this one `beacon_XXXXXXX/vcf`. Inside, you will find `genomicVariationsVcf.json.gz`, the 7th BFF file.

!!! Warning "About disk usage"
    During the annotation process, **multiple intermediate VCF files are created (and kept)**. They're all compressed, but still they will be **as big as your original VCF**. On top of that, `genomicVariationsVcf.json.gz` file can be huge. In summary, **please allocate up to 10x times the space of your original VCF**. Feel free to erase the temporary VCF files `beacon_XXXXXXX/vcf/*vcf.gz` once the job is completed.

Now that you have the 7 JSON files it's time to go to the STEP 3.

## STEP 3

The objective of this step is to load (a.k.a. ingest) the 7 JSON files into **MongoDB**. Once loaded in MongoDB, they are named **collections**.

For doing this we will use again `beacon` script, but this time in mode ***mongodb***.

Let's assume that we have the 6 files from STEP 1 in the directory `my_bff_dir` and the file from STEP 2 at `beacon_XXXXXXX`. 

We will add these values to the parameters file:

```yaml
---
bff:
  metadatadir: my_bff_dir
  # You can change the name of the JSON files
  runs: runs.json
  cohorts: cohorts.json
  biosamples: biosamples.json
  individuals: individuals.json
  analyses: analyses.json
  datasets: datasets.json
  # Note that genomicVariationsVcf is not affected by <metadatadir>
  genomicVariationsVcf: beacon_XXXXXXX/vcf/genomicVariationsVcf.json.gz
```

Finally you execute this command

    ./beacon mongodb -p param_file
         |      |      |
         exe    mode   paramaters file.

If everything goes well, all your data should be loaded into an instance of MongoDB.

![Beacon to MongoDB](img/beacon-mongodb.png)


Congratulations! You can now go to the STEP 4.


## STEP 4

Start making queries with the [API](./api.md).

Cheers!

Manu

!!! Important "Note about MongoDB"
    As with any other database, it is possible to perform queries directly to **MongoDB**. In our case, the database is named _beacon_ and contains the ingested _collections_.    For doing so, you will need to use one of the many UI (we have included [Mongo Express](./external_tools)), the ```mongosh``` or use any of the [MongoDB drivers](https://docs.mongodb.com/drivers) that exist for most programming languages. As an example, we have included an utility `bff-api` that enables you to make **simple queries** to one collection at a time (see instructions [here](https://github.com/EGA-archive/beacon2-ri-tools/tree/main/utils/bff_api)). For a more comprehensive description check [MongoDB](https://www.mongodb.com) literature.

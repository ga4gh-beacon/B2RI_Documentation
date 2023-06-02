# Frequently Asked Questions

## Data ingestion tools

### Are Beacon v2 `genomicVariations.variation.location.interval.{start,end}` coordinates 0-based or 1-based?

They are [0-based](http://docs.genomebeacons.org/formats-standards/#genome-coordinates)

### I have an error when attempting to use `beacon vcf`, what should I do?

* In 9 out 10 cases, the error comes from **BCFtools** and is about the **reference genome** used. The reference genome is set up inside the **parameters** [file](https://github.com/EGA-archive/beacon2-ri-tools) and the possibilities are _hg19_, _hg38_ (both use `chr` before the number), and _hs37_ (does not use `chr` before the number). Be aware that BCFtools is very nit picky (with a reason) about the contigs, etc. not matching those in the fasta file. Please fix your VCF accordingly or modify `config.yaml` to provide the path to your reference genome.

* On top of that, **BCFtools** may complain about the number of fields somewhere (e.g., at INFO) not being right.
```
INFO field IDREP only contains 1 field, expecting 2
```
You can try solving this issue manually or use `bcftools annotate` to get rid of the problematic fields, like this:

    bcftools annotate -x INFO/IDREP input.vcf.gz | gzip > output.vcf.gz

### Can I use SINGLE-SAMPLE and MULTI-SAMPLE VCFs?

Yes, you can use both. MongoDB allows for incremental loads so if you have single sample VCFs that's ok too (you don't need to merge them into a multisample VCF). The connection between sample and variants is performed at the `datasets` collection (and/or `cohorts`).

### Can I use genomic VCF ([gVCF)](https://gatk.broadinstitute.org/hc/en-us/articles/360035531812-GVCF-Genomic-Variant-Call-Format)?

Yes, but **first you will need to transform them** to a VCF. There are quite a few sophisticated ways to do this (e.g., `bcftools convert --gvcf2vcf --fasta ref.fa input.g.vcf`). Here, we are not interested in having the whole genome positions, but rather the positions that contain ALT alleles. A "quick and dirty" solution to get those can be achieved with common Linux tools:

    zcat input.g.vcf.gz | awk '$5 != "<NON_REF>"' | sed 's#,<NON_REF>##' | gzip > output.vcf.gz 

### Why are we re-annotating VCFs | Can I use my own annotations?

The underlying idea about annotating with the B2RI's data ingestion tools is to provide consistency/homogeneicity for the community. Technically speaking, to create `genomicVariationsVcf.json.gz` BFF, first we need to parse an annotated VCF. It's hard to create a parser that handles all the annotation alternatives from the community. On top of that, we need to make sure that the _essential_ fields exist. That's why recommend annotating (or re-annotating if your VCFs already have annotations) VCFs with our tooling. Note that previous annotations in the VCF will be discarded.  

Having said that, in ocassions researchers have **internal annotations** that can have a lot of value as well. For that reason, it is also possible to add alternative genomic variations by filling out the corresponding _tab_ in the provided [XLSX](https://github.com/EGA-archive/beacon2-ri-tools/blob/main/utils/bff_validator/Beacon-v2-Models_template.xlsx). Just make sure you fill out all the mandatory terms requested in the schema. The resulting file will be named `genomicVariations.json`. Both `genomicVariations.json` and `genomicVariationsVcf.json.gz` (see above paragraph) will end up loaded in the MongoDB collection _genomicVariations_. See more information in this [tutorial](./tutorial-data-beaconization.md).

### Is there any alternative to the XLSX to introduce metadata/phenotypic data?

Yes, there is. You can use CSV or JSON directly as input. Please check the [manual](https://github.com/EGA-archive/beacon2-ri-tools/tree/main/utils/bff_validator) of the utility `bff-validator`.

### `bff-validator` specification mismatches

By default, `bff-validator` will validate your data against the default schemas installed with your `beacon2-ri-tools` version. In this regard, it can happen that `bff-validator` gives you warnings on things that look OK [elsewhere](https://github.com/ga4gh-beacon/beacon-v2/issues). An example of this could be warnings on objects matching more than one possibility in `oneOf` keywords. If this happens just use the flag `--ignore-validation` once you are ready to create your `.json` files. 

### Do you load all variations present in a VCF file?

Yes, we do not apply any filter such as using `FILTER` or `QUAL` fields (but we do store those values in case they need to be used _a posteriori_).

### Do you have any recommendations in how to speed up the data ingestion process?

Usually, metadata/phenoclinic data ingestion is fast as we'll be dealing with thousands of values (or dozens of thousands). Processing this should not take more than seconds/minutes.

**VCF processing** is what takes more time, in particular if you have **WGS** (e.g., you can have > 100M variants and thousands of samples). These are some recommendations:

1. Split your VCF by chromosome.

    For this you can use community-tools:
 
        bcftools view input.vcf.gz --regions chr1

    or
    
        tabix -p vcf input.vcf.gz
        tabix input.vcf.gz chr1 | bgzip > chr1.vcf.gz

    or Linux tools (see more [examples](https://bioinformatics.stackexchange.com/questions/3401/how-to-subset-a-vcf-by-chromosome-and-keep-the-header)):

        zcat input.vcf.gz | awk '/^#/ || $1=="chr1"' | bgzip > chr1.vcf.gz

2. Use [parallel processing](https://github.com/EGA-archive/beacon2-ri-tools/tree/main/utils/bff_queue) to submit the jobs. 

### Can I use parallel jobs to perform data ingestion into mongoDB?

Yes, yet this may slow down a bit the ingestion itself.

### When performing incremental uploads, do I need to re-index MongoDB?

Nope. The indexes are created during the first load of key/values and updated automatically on every insert operation. Next attempts for re-indexing are simply [discarded by MongoDB](https://www.mongodb.com/docs/manual/reference/method/db.collection.createIndex/?_ga=2.111853831.262597912.1652622239-541609842.1652622239) (i.e., the operation is idempotent).

### For the CINECA synthetic cohort EUROPE UK1, I could only download the VCF for `chr22` from the installation links. Can I get the VCF for whole genome?

Yes, but you need to request access and download it from the [EGA](https://ega-archive.org/datasets/EGAD00001006673). The VCF file contains WGS for 2,504 fake individuals (~20G). See more information [here](./synthetic-dataset.md). 

## Beacon v2 API 

### Which tool do you recommend for making queries?

We recommend [Curl](https://curl.se).

### Are aternative schemas supported?

_A priori_, [Beacon v2](http://docs.genomebeacons.org/) specification allows for alternative schema for the responses (e.g., [Phenopackets](https://phenopacket-schema.readthedocs.io/en/latest)). At this time (Apr-2022), this option is not supported by the Beacon v2 API.

### Is the response data encrypted?

By default, the API will start as `http`. The request/response can be encripted by adding the `https` protocol on top.

## B2RI (General)

### Is B2RI free?

**Yes**, it is open sourc and it is free. The data ingestion tools have [GNU General Public License v3.0](https://en.wikipedia.org/wiki/GNU_General_Public_License#Version_3) and the API has [Apache License v2.0](https://www.apache.org/licenses/LICENSE-2.0). 

The included [CINECA_synthetic_cohort_EUROPE_UK1](https://www.cineca-project.eu/cineca-synthetic-datasets) dataset has [CC-BY](https://en.wikipedia.org/wiki/Creative_Commons_licens) license.

### Does it come with an UI (user interface)? 

The simple answer is no. All tools are `command-line` based. But, we added a [browser](./data-ingestion.md#bff-genomic-variations-browser) to visualize the contents of `genomicVariations` collection. This simple tool will (likely) cover many of the typical needs of a clinical geneticist/bioinformatician.

The long answer is that at [CRG](https://www.crg.eu/) we're working on a full UI to work on top of the REST API. 

### Does B2RI include tools to create a Beacon Network?

Nope at this moment (Apr-2022). Currently, there is a Beacon scout team actively developing Beacon v2 Network API specification. See [Beacon v1 Network API specification](https://github.com/CSCfi/beacon-network) as a reference.

### I am using a SQL-based database, can I still use your Reference Implementation?

The issue with having a SQL-based is that, if you want to be **Beacon v2 response _compliant_** you will need to convert your tables-based data to the JSON Schema of the [Beacon v2 Models](http://docs.genomebeacons.org). Intrepid implementers are able to do this transformation (likely) at the API level. However, a much simpler alternative (and actually the one we've seen the most in healthcare systems) is that people perform a `dump` (data export) of the subset of data they want to share and then use [B2RI tools](tutorial-data-beaconization.md) to convert this tabular data to Beacon v2 format and use the included REST API. So yes, if you follow this path you will be still using our Reference Implementation.

### Should I update to the `latest` version?

Yep. We recommend checking our Github repositories ([beacon2-ri-tools](https://github.com/EGA-archive/beacon2-ri-tools) and [beacon2-ri-api](https://github.com/EGA-archive/beacon2-ri-api)) and downloading the latest version. If the version it's in GitHub is because it passed our test and it's ready to be used. In principle, a simple `git pull` will do the update for both containerized and non-containerized versions.
Now, the version number matches that of the [Beacon v2 specification](https://github.com/ga4gh-beacon/beacon-v2). When the latter changes, B2RI version will change as well.

### Can I contribute to the GitHub repositories?

Please contact the [authors](./about.md).

### How do I cite B2RI?

!!! Note "Citation"
    Rueda, M, Ariosa R. "Beacon v2 Reference Implementation: a toolkit to enable federated sharing of genomic and phenotypic data". _Bioinformatics_, btac568, [DOI](https://doi.org/10.1093/bioinformatics/btac568).

# GDI Starter Kit deployment guide [DRAFT]

This guide provides a set of instructions on how to deploy the GDI Starter Kit
in a node. It also includes the necessary steps to upload data and connect to
the central services.


## Prerequisites

- [Docker](https://docs.docker.com/engine/install/)
- [Docker Compose](https://docs.docker.com/compose/install/)



## Deployment

### FAIR Data Point

https://docs.fairdatapoint.org/en/latest/deployment/local-deployment.html

### Beacon V2

#### Requirements

System and services:
- OS: UNIX (Linux, Mac…)
- Architecture: amd64/arm64

Hardware:
- CPU: > 4
- RAM: > 4 GB (generally, 20% of total data size as RAM, e.g. 40 GB -> 8 GB RAM)
- Disk limit: > 32 GB

Software:
- Docker engine: version > 20.10.18


#### Aggregated Beacon instance

First, clone the beacon repository. You may do it in two ways:

- Clone the original repository and checkout to the stable branch:
```bash
git clone https://github.com/EGA-archive/beacon2-pi-api.git
cd beacon2-pi-api
git checkout main
```

- Clone the starter-kit repository and load the beacon2-pi-api submodule:

```bash
git clone https://github.com/GenomicDataInfrastructure/starter-kit.git
cd starter-kit
git submodule update --init beacon-v2/
```

Edit the config file `ri-tools/conf/conf.py` by changing these variables:

- `datasetId`: this variable has to match the “id” of the dataset you will relate the
variants to.
- `case_level_data`: change it to False.
- `reference_genome`: select your genome of reference between NCBI36, GRCh37 and GRCh38.

Edit the file `ri-tools/pipelines/default/templates/populations.json` and change the variable
`numberOfPopulations` to the exact number of ancestries/populations you have allele frequencies for in your VCF and map
how are the allele frequencies, zygosities and name of the population annotated in your VCF for each population.

Make sure the next list of ports are free of use in your system:

- 27017 (MongoDB)
- 5050 (Beacon)

Light up the needed containers from the deploy folder:

```bash
docker compose up -d --build beaconprod db beacon-ri-tools
```

If the containers are built correctly the Beacon API will run in http://localhost:5050/api


##### Data injection

Copy your VCF files in .gz format inside the folder `ri-tools/files/vcf/files_to_read/`. Then, inject the variant data
from the VCFs executing the next command (this step may take a few hours to finish, depending on your system resources):

```bash
docker exec ri-tools python genomicVariations_vcf.py
```

Inject the phenotypic data replacing `path/to/datasets.json` with the correct path to your file:

```bash
docker cp path/to/datasets.json mongoprod:tmp/datasets.json
docker exec mongoprod mongoimport --jsonArray --uri "mongodb://root:example@127.0.0.1:27017/beacon?authSource=admin" --file /tmp/datasets.json --collection datasets
```

Edit the file `beacon/permissions/datasets/datasets_permissions.yml` adding the dataset id and its permissions. For
example:

```yaml
dataset_id:
    public:
        default_entry_types_granularity: record
```

For the API to respond fast to the queries, you have to index your database each time you inject new data::

```bash
docker exec beaconprod python -m beacon.connections.mongo.reindex
```

If your data collections (e.g., runs, biosamples, etc.) already contain structured metadata using ontology terms
(like NCIT, UBERON, EFO...), you can extract filtering terms automatically. This will populate the `/filteringTerms` endpoint of your Beacon, enabling more advanced queries:

```bash
docker exec beaconprod python -m beacon.connections.mongo.extract_filtering_terms
```


#### Customization

Make your Beacon your own by following these next steps:

- Edit your instance’s [metadata](https://beacon-documentation-demo.ega-archive.org/configuration#editing-beacon-info). Update the `/info` endpoint with your organization's name, description, version, and contact details.
- Manage dataset [permissions](https://beacon-documentation-demo.ega-archive.org/configuration#managing-dataset-permissions). Control which datasets are public or require authentication.
- Enable advanced [filtering](https://beacon-documentation-demo.ega-archive.org/filtering-terms#extract-terms).
- Found more setting options in [Configuration](https://beacon-documentation-demo.ega-archive.org/configuration).


#### Connection to the GDI Beacon Network

Follow this steps:

- Validate the schema of your Beacon instance with [Beacon Verifier v2](https://beacon-verifier-demo.ega-archive.org/).
- Add your instance to the test Beacon Network by contacting [Oriol Lopez-Doriga](#contact) to validate its configuration
- Upon successful testing, the Beacon instance will be added to the GDI Beacon Network by the CRG Beacon team.

#### References

- [Guidelines](https://docs.google.com/document/d/1nytWC6QOvaLmoJd0OBOENc52gEXagXzI).
- [Product Documentation](https://beacon-documentation-demo.ega-archive.org/).
- [Protocol Documentation](https://docs.genomebeacons.org/).

#### Contact

Beacon team at the Centre for Genomic Regulation (CRG), Barcelona, Spain:

- Jordi Rambla (team lead): jordi.rambla@crg.eu.
- Liina Nagirnaja (Beacon manager): liina.nagirnaja@crg.eu.
- Oriol Lopez-Doriga (Beacon v2 API developer): oriol.lopezdoriga@crg.eu. 

### LS AAI

### REMS

### S&I

### Compute

### htsget


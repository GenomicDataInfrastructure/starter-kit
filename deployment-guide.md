# GDI Starter Kit deployment guide [WIP]

This guide provides a set of instructions on how to deploy the GDI Starter Kit
in a node. It also includes the necessary steps to upload data and connect to
the central services.


## Prerequisites

- [Docker](https://docs.docker.com/engine/install/)
- [Docker Compose](https://docs.docker.com/compose/install/)



## Deployment

### FAIR Data Point

#### Installation

First, create a docker compose file (e.g. `compose.yml`) as this in an empty directory:

```yaml
 # compose.yml

services:
  fdp:
    image: "fairdata/fairdatapoint:${FDP_VERSION:-1.18}"
    restart: no
    environment:
      SERVER_PORT: 8080
      INSTANCE_CLIENTURL: http://localhost
      INSTANCE_PERSISTENTURL: http://localhost
    depends_on:
      mongo:
        condition: service_healthy
      graphdb:
        condition: service_healthy
    healthcheck:
      test: wget --quiet --spider http://127.0.0.1:8080 || exit 1
      start_interval: 3s
      start_period: 30s

  fdp-client:
      image: "fairdata/fairdatapoint-client:${FDP_CLIENT_VERSION:-1.18}"
      restart: no
      ports:
        - "80:80"
      environment:
        FDP_HOST: fdp:8080
      depends_on:
        fdp:
          condition: service_healthy
      healthcheck:
        test: wget --quiet --spider http://127.0.0.1 || exit 1
        start_interval: 3s
        start_period: 30s

  mongo:
    image: "mongo:${MONGO_VERSION:-8.0}"
    restart: no
    healthcheck:
      test: |
        [ $(mongosh --quiet --host localhost:27017 --eval "db.runCommand('ping').ok") = 1 ] || exit 1
      start_interval: 3s
      start_period: 30s

  graphdb:
    image: ontotext/graphdb:10.7.6
    volumes:
        - ./graphdb:/opt/graphdb/home
        - ./repo.json:/tmp/repo.json:ro
    entrypoint:
        - bash
        - -c
        - |
          # enable bash job control
          set -m

          # start graphdb and move it to the background
          /opt/graphdb/dist/bin/graphdb &

          # wait for 10 sec
          sleep 10

          # create the repository
          curl -X POST http://localhost:7200/rest/repositories -H "Content-Type: application/json" -d "@repo.json"

          # move graphdb job to foreground
          fg
    healthcheck:
        # https://graphdb.ontotext.com/documentation/10.7/database-health-checks.html
        test: curl --fail-with-body http://localhost:7200/repositories/fdp/health || exit 1

```

Then, create a file `application.yml` next to it as this:

```yaml
# application.yml
repository:
    type: 4
    graphDb:
        url: http://graphdb:7200
        repository: fdp
```

And another file named `repo.json` containing the configuration for the GraphDB repository:

```json
{
    "id": "fdp",
    "type": "graphdb",
    "params": {
        "title": {
            "label": "Repository description",
            "name": "",
            "value": ""
        },
        "defaultNS": {
            "label": "Default namespaces for imports(';' delimited)",
            "name": "defaultNS",
            "value": ""
        },
        "imports": {
            "label": "Imported RDF files(';' delimited)",
            "name": "imports",
            "value": ""
        }
    }
}
```

You can now run it using:

```bash
docker compose up -d
```

After some time, you should be able to access FDP at http://localhost.

There are two default user accounts, that you should change once your FDP becomes publicly available:

- albert.einstein@example.com with password `password` and role `admin` 
- nikola.tesla@example.com with password `password` and role `user`

Here you can find a way to set up a public instance of FDP using a reverse proxy:
https://docs.fairdatapoint.org/en/latest/deployment/production-deployment.html.

#### Connection to User Portal

Once you have deployed FDP, please communicate the host URL of your instance with LNDS team.


#### Metadata onboarding

To add GDI-specific SHACL validation, follow these steps to register the SHACL shapes: 

##### Step 1: Download SHACL Shapes
   - Access the GDI-specific SHACL shapes from this [GDI metadata repository](https://github.com/GenomicDataInfrastructure/gdi-metadata/tree/main/Formulasation(shacl)/core/PiecesShape).
   - Download each SHACL shape file (e.g., `Resource.ttl` and others).

##### Step 2: Upload SHACL Shapes to FDP

1. **Login** to FDP using an admin account.
2. Navigate to **Metadata Schemas** (located in the dropdown under your username).
3. For each SHACL shape file you downloaded:
   - Open the **editor** and paste the contents of `Resource.ttl` (or other shapes).
   - **Add a description** to document the purpose or release information.
   - Configure the following:
      - ✅ **For `Resource.ttl`:** check **Abstract** (this shape is a base class).  
      - 🚫 **For all other shapes:** leave **Abstract** unchecked.
   - Click **Save and Release** to finalize the shape.
   - Provide a meaningful description and **version number** for the release.
   - Check the **public** checkbox to make the shape accessible.
   - Click **Release** to complete the upload.

Repeat these steps for each SHACL shape file.

##### Large datasets 
To onboard large datasets more efficiently, you can use a Jupyter notebook to automate this process.
Clone the [Sempyro repository](https://github.com/Health-RI/SeMPyRO) and run the notebook using:

```bash
hatch run docs:jupyter lab
```

Then, execute the [GDI-specific notebook](https://github.com/Health-RI/SeMPyRO/blob/main/docs/Usage_example_GDI.ipynb)
to complete the upload.

#### Contact

For questions or support, please contact:

- **E-mail** – [servicedesk@health-ri.nl](mailto:servicedesk@health-ri.nl)

#### References

- [Product documentation](https://docs.fairdatapoint.org/en/latest/).
- [Specification](https://specs.fairdatapoint.org/fdp-specs-v1.2.html).
- [Integration with User Portal](https://genomicdatainfrastructure.github.io/gdi-userportal-docs/developer-guide/fdp/).

### Beacon V2

#### Requirements

System and services:
- OS: UNIX (Linux, Mac…)
- Architecture: amd64/arm64

Hardware:
- CPU: >= 4
- RAM: >= 4 GB (generally, 20% of total data size as RAM, e.g. 40 GB -> 8 GB RAM)
- Disk limit: >= 32 GB

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
cd beacon-v2
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

Inject the phenotypic data replacing `datasets.json` with the correct path to your file:

```bash
gzip datasets.json
gunzip --stdout datasets.json.gz | docker exec -i mongoprod sh -c 'mongoimport --jsonArray --uri "mongodb://root:example@127.0.0.1:27017/beacon?authSource=admin" --collection datasets'
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
- Add your instance to the test Beacon Network by contacting [Oriol Lopez-Doriga](#contact) to validate its configuration.
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

For connecting your services to [LS AAI](https://lifescience-ri.eu/ls-login.html) read the [Documentation for connecting your service to LS AAI](https://lifescience-ri.eu/ls-login/documentation/service-provider-documentation/service-provider-documentation.html), mainly the [Instructions for relying parties](https://docs.google.com/document/d/17pNXM_psYOP5rWF302ObAJACsfYnEWhjvxAHzcjvfIE/edit?usp=sharing).

The main user interface to LS AAI for users is [LS AAI User Profile](https://profile.aai.lifescience-ri.eu/)

The main user interface for Service Provider administrators is [Service Provider Registry](https://services.aai.lifescience-ri.eu/spreg/)

The main user interface for group administrators is [LS Identity and Access Management](https://perun.aai.lifescience-ri.eu/)

### REMS

### S&I

### Compute

### htsget


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

First, clone the beacon repository. You may do it in two ways:

- Clone the original repository and checkout to the stable branch:
> [TO CHECK] Set stable branch
```bash
git clone https://github.com/EGA-archive/beacon2-pi-api.git
cd beacon2-pi-api
git checkout main
```

- Clone the starter-kit repository and load the beacon2-pi-api submodule:

```bash
git clone https://github.com/GenomicDataInfrastructure/starter-kit.git
cd starter-kit/beacon-v2
git submodule init
git submodule update
```

> [TO CHECK] Is this needed?
> 
> [Data from Beacon RI Tools v2](https://github.com/EGA-archive/beacon2-ri-tools-v2). Please, bear in mind that the datasetId for your records must match the id for the dataset in the /datasets entry type

Make sure the next list of ports are free of use in your system:

- 27017 (MongoDB)
- 5050 (Beacon)

> [TO CHECK] These are not in https://beacon-documentation-demo.ega-archive.org/pi-automated-deployment but are in 
the docker-compose.yml file and also mentioned in https://beacon-documentation-demo.ega-archive.org/pi-automated-deployment
- 8081 (mongo-express) 
- 8080 (Keycloak)
- 9991 (Keycloak SSL)


To quickly deploy your Beacon instance and load initial (test) data, run the following command from the root of your project:

```bash
bash mongostart.sh
```

If the operation is successful, you will have a beacon up and running at http://localhost:5050/api.

> [TO CHECK] How is this done?
>
> If you want to use your own data, simply replace the contents of the data folder with your custom Beacon Friendly Format (BFF) files before running the script.

> [TO CHECK] When is this step needed? Just after the `mongostart.sh`? It is not present in the production documentation
but it is in the production beacon README.
>
> For the API to respond fast to the queries, you have to index your database. You can create the necessary indexes by running the next script:

```bash
docker exec beaconprod python -m beacon.connections.mongo.reindex
```

Note that you will need to run this script each time you inject new data.

#### Customization
Make your Beacon your own by following these next steps:
- Edit your instance’s [metadata](https://beacon-documentation-demo.ega-archive.org/configuration#editing-beacon-info). Update the /info endpoint with your organization's name, description, version, and contact details.
- Manage dataset [permissions](https://beacon-documentation-demo.ega-archive.org/configuration#managing-dataset-permissions). Control which datasets are public or require authentication.
- Enable advanced [filtering](https://beacon-documentation-demo.ega-archive.org/filtering-terms#extract-terms).
- Found more setting options in [Configuration](https://beacon-documentation-demo.ega-archive.org/configuration)


#### References
- Guidelines: https://docs.google.com/document/d/1nytWC6QOvaLmoJd0OBOENc52gEXagXzI
- Documenation: https://beacon-documentation-demo.ega-archive.org/

### LS AAI

### REMS

### S&I

### Compute

### htsget


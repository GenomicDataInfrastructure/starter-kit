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

Make sure the next list of ports are free of use in your system:

- 27017 (MongoDB)
- 5050 (Beacon)
- 8081 (mongo-express) 
- 8080 (Keycloak)
- 9991 (Keycloak SSL)


Light up the containers from the deploy folder:

```bash
docker compose up -d --build
```

If the containers are built correctly:

- The Beacon API will run in http://localhost:5050/api.
- The mongo-express UI will run in http://localhost:8081.
- The Keycloak UI will run in http://localhost:8080/auth.


#### Data injection

If you already have BFF files, copy them into the [data folder for Mongo database](https://github.com/EGA-archive/beacon2-pi-api/tree/main/beacon/connections/mongo/data). Then execute this command to insert the files into the database:

```bash
for file in /data/*.json; do
    collection=$(basename "$file" .json)
    docker exec mongoprod mongoimport --jsonArray \
        --uri "mongodb://root:example@127.0.0.1:27017/beacon?authSource=admin" \
        --file "$file" --collection "$collection"
done
```

For the API to respond fast to the queries, you have to index your database. You can create the necessary indexes by running the next script:

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


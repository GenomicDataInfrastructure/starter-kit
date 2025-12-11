# GDI Starter Kit deployment guide [DRAFT]

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

name: fdppv1

services:
  graphdb:
    image: "ontotext/graphdb:${GRAPHDB_VERSION:-10.8.8}"
    restart: no
    ports:
      - "127.0.0.1:7200:7200"
    volumes:
      - graphdbdata:/opt/graphdb/home
    healthcheck:
      test: curl http://localhost:7200/rest/monitor/infrastructure --silent --fail || exit 1
      start_interval: 3s
      start_period: 30s

  graphdb-init:
    image: "curlimages/curl:${CURL_VERSION:-latest}"
    restart: no
    environment:
      REPO_FILE: /repo.json
      REPO_NAMES: fdp
    volumes:
      - ./repo.json:/repo.json:ro
    # the following curl request returns status 201 if the repo does not exist, or status 400 otherwise
    command:
      - sh
      - -c
      - |
        for name in $$REPO_NAMES
        do
        # substitute "repo-name" by variable name in graphdb repo template
        data=$$(sed s/repo-name/$$name/g /repo.json)
        # post repo definition to graphdb rest api
        curl http://graphdb:7200/rest/repositories --verbose --silent --header "Content-Type: application/json" --data "$$data"
        done
    depends_on:
      graphdb:
        condition: service_healthy

  mongo:
    image: "mongo:${MONGO_VERSION:-8.0}"
    restart: no
    volumes:
      - mongodbdata:/data/db
    healthcheck:
      test: |
        [ $(mongosh --quiet --host localhost:27017 --eval "db.runCommand('ping').ok") = 1 ] || exit 1
      start_interval: 3s
      start_period: 30s

  fdp:
    image: "fairdata/fairdatapoint:${FDP_VERSION:-1.18}"
    restart: no
    environment:
      SERVER_PORT: 8080
      INSTANCE_CLIENTURL: http://localhost #CHANGE_ME BEFORE DOCKER COMPOSE UP TO PREFERED CUSTOM DOMAIN 
      INSTANCE_PERSISTENTURL: http://localhost #CHANGE_ME BEFORE DOCKER COMPOSE UP TO PREFERED CUSTOM DOMAIN 
      REPOSITORY_TYPE: 4
      REPOSITORY_GRAPHDB_URL: http://graphdb:7200
      REPOSITORY_GRAPHDB_REPOSITORY: fdp
    depends_on:
      mongo:
        condition: service_healthy
      graphdb-init:
        condition: service_completed_successfully
    healthcheck:
      test: wget --quiet --spider http://127.0.0.1:8080 || exit 1
      start_interval: 3s
      start_period: 30s

  fdp-client:
    image: "fairdata/fairdatapoint-client:${FDP_CLIENT_VERSION:-1.18}"
    restart: no
    ports:
      - "127.0.0.1:80:80"
    environment:
      FDP_HOST: fdp:8080
    depends_on:
      fdp:
        condition: service_healthy
    healthcheck:
      test: wget --quiet --spider http://127.0.0.1 || exit 1
      start_interval: 3s
      start_period: 30s

volumes:
  graphdbdata:
  mongodbdata:

```

And another file named `repo.json` containing the configuration for the GraphDB repository:

```json
{
  "id": "fdp",
  "title": "",
  "type": "graphdb",
  "params": {
    "defaultNS": {
      "name": "defaultNS",
      "label": "Default namespaces for imports(';' delimited)",
      "value": ""
    },
    "imports": {
      "name": "imports",
      "label": "Imported RDF files(';' delimited)",
      "value": ""
    }
  }
}
```

You can now run it using:

```bash
docker compose up -d
```

After some time, you should be able to access FDP at http://localhost or your prefered custom domain.

There are two default user accounts, that you should change once your FDP becomes publicly available:

- albert.einstein@example.com with password `password` and role `admin` 
- nikola.tesla@example.com with password `password` and role `user`

Here you can find a way to set up a public instance of FDP using a reverse proxy:
https://docs.fairdatapoint.org/en/latest/deployment/production-deployment.html.

#### Connection to User Portal

Once you have deployed FDP, please communicate the host URL of your instance with LNDS team.


#### Metadata onboarding

##### Metadata validation

To onboard your data according to the latest GDI harmonised metadata model, you need to update the SHACLs.
Go to [GDI metadata](https://github.com/GenomicDataInfrastructure/gdi-metadata) to update them.


##### Large datasets 
To onboard large datasets more efficiently, you can use a Jupyter notebook to automate this process.
Clone the [Sempyro repository](https://github.com/Health-RI/SeMPyRO) and run the notebook using:

```bash
hatch run docs:jupyter lab
```

Then, execute for example the [GDI-specific notebook](https://github.com/Health-RI/SeMPyRO/blob/main/docs/Usage_example_GDI.ipynb)
to complete the upload.

#### Contact

For questions or support, please contact:

- **E-mail** – [servicedesk@health-ri.nl](mailto:servicedesk@health-ri.nl)

#### References

- [Product documentation](https://docs.fairdatapoint.org/en/latest/).
- [Specification](https://specs.fairdatapoint.org/fdp-specs-v1.2.html).
- [Integration with User Portal](https://genomicdatainfrastructure.github.io/gdi-userportal-docs/developer-guide/fdp/).

### Beacon V2

### LS AAI

### REMS

### S&I

### Compute

### htsget


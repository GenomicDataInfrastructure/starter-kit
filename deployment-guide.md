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
    ports:
        - 7200:7200
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

After some time, you should be able to access FDP at http://localhost:8080.

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

### LS AAI

### REMS

### S&I

### Compute

### htsget


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

First, create a docker compose file (e.g. `compose.yml`) as this:

```yaml
 # compose.yml

 services:

     fdp:
         image: fairdata/fairdatapoint:1.16
         volumes:
             - ./application.yml:/fdp/application.yml:ro

     fdp-client:
         image: fairdata/fairdatapoint-client:1.16
         ports:
             - 80:80
         environment:
             - FDP_HOST=fdp

     mongo:
         image: mongo:4.0.12
         ports:
             - 27017:27017
         volumes:
             - ./mongo/data:/data/db

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
        interval: 5s
```

Then, create a file `application.yml` as this:

```yaml
# application.yml

instance:
    clientUrl: http://localhost:80 # 80 is the default exposed port. Change it in compose.yml if you change it here.

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

After some time, you should be able to access FDP at http://localhost:80.

There are two default user accounts, that you should change once your FDP becomes publicly available:

- albert.einstein@example.com with password `password` and role `admin` 
- nikola.tesla@example.com with password `password` and role `user`

Here you can find a way to set up a public instance of FDP using a reverse proxy:
https://docs.fairdatapoint.org/en/latest/deployment/production-deployment.html.

#### Connection to User Portal

Once you have deployed FDP, please communicate the host URL of your instance with LNDS team.


#### Metadata onboarding

To onboard large datasets more efficiently, you can use a Jupyter notebook to automate this process.
Clone the [Sempyro repository](https://github.com/Health-RI/SeMPyRO) and run the notebook using:

```bash
hatch run docs:jupyter lab
```

Then, execute the [GDI-specific notebook](https://github.com/Health-RI/SeMPyRO/blob/main/docs/Usage_example_GDI.ipynb)
to complete the upload.

#### Contact

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


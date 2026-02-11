# GDI Starter Kit Repository

The European Genomic Data Infrastructure (GDI) project aims to enable federated and secure access to genomic and related phenotypic and clinical data to improve research, policymaking and healthcare across Europe. This will create unprecedented opportunities for transnational and multi-stakeholder actions in personalised medicine for cancer, common, rare and infectious diseases as well as access to a reference genome collection representing the European population (Genome of Europe).

GDI will develop and deploy the technical capacity needed to realise, scale, and sustain this effort, aiming to help at least 15 countries achieve a fully operational infrastructure by 2026. This repository contains a collection of software applications and components based on open community standards from the Global Alliance for Genomics and Health (GA4GH). By employing this Starter Kit, nodes can acquire that technical ability to access genomic and phenotypic data across borders.

Overall, the GDI Starter Kit provides the following functionalities: data discovery, data access management, data storage, data reception and data processing. The infrastructure consists of central (European level) and local (node level) components, which are described below.

## Functionalities and Components

### Central components

- **Beacon Network - Data discovery**: A federated network of beacon nodes, which are responsible for the discovery of genomic variants and biomedical data across distributed resources.

- [**LifeScience Authentication and Authorisation Infrastructure (LS-AAI) - Authentication and authorization**](https://services.aai.lifescience-ri.eu/): An authentication service implementing the OpenID Connect protocol and providing federated identity and access control.

- [**User Portal - Data discovery and authorization**](https://portal.gdi.lu/): A web application that serves as a data catalogue and allows users to manage their access rights to resources such as research datasets.


### Local components

- **Beacon - Data discovery**: An implementation of the GA4GH Beacon protocol, an open standard that enables the discovery of genomic variants and biomedical data across single or distributed resources. There are two instances of this component:

    - **Aggregated Beacon**: Supports aggregated data without processing personal data.

    - **Subject Level Beacon**: Supports genomic and phenotypic individual-level data.

- **FAIR Data Point (FDP) - Data discovery**: A metadata service that provides access to metadata following the FAIR principles.

- **Funnel - Data processing**: An implementation of the GA4GH Task Execution Schemas (TES), aiming to standardize APIs used for task execution across multiple platforms.

- **htsget - Data reception**: An implementation of the htsget protocol, enabling secure, efficient, and reliable retrieval of sequencing read and variation data.

- **Resource Entitlement Management System (REMS) - Authorization**: A tool to manage access rights to resources such as research datasets, ensuring data security and privacy.

- **Sensitive Data Archive (SDA) - Data storage**: A stack of services for storage and controlled access to sensitive data.

- **Synthetic data**: A set of genomic and phenotypic synthetic data and metadata that can be used for testing and development purposes.

Please note that these components serve as an example of a feasible implementation and may be replaced with equivalent software pieces complying with the GA4GH standards and the expected use cases. They should also be developed following best practices and security guidelines, such as data protection by design and by default.

## APIs
In this other repo you will find the links to the API specifications, and/or associated standard specifications.
https://github.com/GenomicDataInfrastructure/api-links

## Deployments

The following repositories document how some nodes have deployed these components:

- [Sweden MS7 2023 Deployment Notes](https://github.com/GenomicDataInfrastructure/starter-kit-se-deployment-notes).
- [Luxembourg MS7 2023 Deployment Notes](https://github.com/GenomicDataInfrastructure/starter-kit-lu-deployment).
- [Germany Deployment Notes](https://github.com/GenomicDataInfrastructure/starter-kit-de-deployment-notes).
- [Estonia Deployment Notes](https://gdi.ut.ee/en/technical/deployment-notes/).

Estonian node also developed a standalone, minimalistic node implementation covering FDP and Beacon: https://github.com/GenomicDataInfrastructure/gdi-mini-node.

## Disclaimer
For now, these components are intended for development and testing purposes only. They should not be used in production environments as they may contain bugs, security vulnerabilities or other issues that could impact data integrity and confidentiality. For this reasons, only synthetic data should be used with these components. Later steps will be focused on the development of a production-ready solution.


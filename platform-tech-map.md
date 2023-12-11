### External service

- platform-api
  - Scala - Sangria (GrahpQL) - Play Framework (web framework)
  - SKSamuel elastic 4s (Elastics search binding)
  - Clickhouse JDBC
- ot-ai-api
  - NodeJS - Express (web framework)
  - Langchain (framework for LLMs integration)
- ot-ui-apps
  - Genetics/Platform UI
  - Javascript/Typescript
  - React - ViteJS to bundle - MUI (UI framework)

### Internal tool

- Platform input support
  - Python - CLI tool
  - sources (any provided by data team):
    - Google buckets
    - FTP
    - HTTP
    - Elastic search
  - manifest
    - metadata (info about input and process)
- ETL backend
  - Scala - Spark (framework for ETL, .jar)
  - produces json/parquet
  - config file
- Platform output support
  - CLI tool
    - Shell scripts
  - Infrastructure capabilities
  - Generates statics based on metadata (UI assets)
  - Loads ETL output to Data backends (data images)
  - Loads ETL output to FTP/BigQuery
  - Loads ETL output to Prod data buckets
- Infrastructure definition
  - Terraform (infra as code)
    - Configured for Google cloud

* - stand alone deployment
  - Docker compose - Shell scripts
    - local elastic search
    - local api
    - loca UI
    - local ot-ai-api
    - local clickhouse

### Data backend

- Virtual machines
  - Elastic search v7.x / Open search **_new_**
  - Clickhouse
- Backend as service (GCP)
  - BigQuery

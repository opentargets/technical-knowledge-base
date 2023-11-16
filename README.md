# Platform Technical Documentation

- How to run the indivdual stages:
  - [platform input support](platform-input-support.md)
  - [etl](platform-etl-backend.md)
  - [platform output support](platform-output-support.md)
  - [data/software deployment](terraform-google-open-targets-platform.md)

  

## Generate the data

#### *DEV* or *PROD*

1. Get the data and stage it for the ETL by running the entire [platform input support](platform-input-support.md)
2. Transform the data by running the entire [etl](platform-etl-backend.md)
3. [Make data images](platform-output-support.md#make-the-data-images)

#### *DEV-PPP* or *PPP* 
tbc

## Deploy data/software to production
- Deploying data or software is conceptually the same with terraform.
- Deploying data requires the data images to be built - see [above](#generate-the-data). 
- Deploying software - except for the webapp, requires the images to be built (see individual repos) in advance. In terraform you reference the specifc image/tag in the terraform config to deploy that version of the data/software.

#### *DEV*
1. Setup the dev profile in the [terraform repo](terraform-google-open-targets-platform.md) and execute.

#### *PROD*
1. If releasing data - do the release data step from [POS](platform-output-support.md#release-data)
2. Setup the prod profile in the [terraform repo](terraform-google-open-targets-platform.md) and execute

#### *DEV-PPP*
#### *PPP* 

## Deploy an updated component e.g. data image, webapp or the API
1. Exactly as [above](#deploy-datasoftware-to-production) but leave all variables in the profile as they were except the ones you want to modify. E.g. if you _only_ want to bump the webapp version, just change the version number for the webapp in the config.
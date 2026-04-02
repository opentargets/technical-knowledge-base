---
icon: lucide/rocket
---

# Release Process


---

!!! note
    OLD DOCS BELOW


- How to run the indivdual stages:
  - [platform input support](platform-input-support.md)
  - [etl](platform-etl-backend.md)
  - [platform output support](platform-output-support.md)
  - [data/software deployment](terraform-google-open-targets-platform.md)



## Generate the data

#### *PROD*

1. Get the data and stage it for the ETL by running the entire [platform input support](platform-input-support.md)
2. Transform the data by running the entire [etl](platform-etl-backend.md)
3. [Make data images](platform-output-support.md#make-the-data-images)

#### *DEV*, *DEV-PPP* or *PPP*
to follow

## Deploy data/software to production
- Deploying data or software is conceptually the same with terraform.
- Deploying data requires the data images to be built - see [above](#generate-the-data).
- Deploying software - except for the webapp, requires the images to be built (see individual repos) in advance. In terraform you reference the specifc image/tag in the terraform config to deploy that version of the data/software.

#### *DEV*
1. Setup the dev profile in the [terraform repo](terraform-google-open-targets-platform.md) and execute.

#### *PROD*
1. If releasing data - do the release data step from [POS](platform-output-support.md#release-data)
2. Setup the prod profile in the [terraform repo](terraform-google-open-targets-platform.md) and execute

#### *DEV-PPP* or *PPP*
to follow

## Deploy an updated component e.g. data image, webapp or the API
1. Exactly as [above](#deploy-datasoftware-to-production) but leave all variables in the profile as they were except the ones you want to modify. E.g. if you _only_ want to bump the webapp version, just change the version number for the webapp in the config.



# Platform Input Support

[Repo](https://github.com/opentargets/platform-input-support)

## Platform release

### Build

We commit the configuration for each release to the public repo. If there

- _Clone_ the repo
- _Checkout_ a release branch from `master` e.g. `release_23-12`
- _Edit_ the `./config.yaml`
  - Update versions for **_EFO_**, **_Ensembl_** and **_CHEMBL_** - these versions are decided by the data team
    - **EFO**: one place to update
    - **CHEMBL**: several places to update
    - **Ensembl**: several places to update
- _Commit_ and _push_ to public repo - this triggers a [Travis build](https://app.travis-ci.com/github/opentargets/platform-input-support)
- _Wait_ for the image to be published on [quay](https://quay.io/repository/opentargets/platform-input-support?tab=tags)

### Run PIS

#### Setup GCP infrastructure

_Note: this could be done by terraform_

- _Create_ a [compute engine](https://console.cloud.google.com/compute/instancesAdd?cloudshell=false&project=open-targets-eu-dev) (vm) with the following specs
  - e2-standard-4
  - a boot disk with at least 500GB

#### Install dependencies

- _ssh_ the machine `gcloud compute ssh --zone <ZONE> <VM_NAME> --project "open-targets-eu-dev"
`
- run the following (change the `IMAGE_TAG` and `RELEASE_VERSION` accordingly)

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce tmux
sudo usermod -a -G docker $USER
newgrp docker
mkdir -m 775 -p opentargets/credentials
mkdir -m 775 -p opentargets/output
mkdir -m 775 -p opentargets/log
gsutil cp gs://open-targets-ops/credentials/pis-service_account.json opentargets/credentials/open-targets-gac.json
```

#### Run the PIS container

```bash
tmux new -s pisrun

# Set image and release versions
IMAGE_TAG="release_23-12"
RELEASE_VERSION="23.12"

docker run -v /home/$USER/opentargets/output:/srv/output -v /home/$USER/opentargets/log:/usr/src/app/log -v /home/$USER/opentargets/credentials/open-targets-gac.json:/srv/credentials/open-targets-gac.json quay.io/opentargets/platform-input-support:$IMAGE_TAG -o /srv/output --log-level=DEBUG -gkey /srv/credentials/open-targets-gac.json -gb open-targets-pre-data-releases/$RELEASE_VERSION/input -exclude otar pppevidence
```

#### Final Steps

- Check the files are copied to the bucket `gs://open-targets-pre-data-releases/$RELEASE_VERSION/input`
- Check the manifest file to see all the steps ran correctly. You can do that using `jq 'del(.steps)' manifest.json`. If there's any errors you can do additional checks using the following commands:
  - `jq '' manifest.json` : query all information
  - `jq 'del(.steps)' manifest.json` : query the header with no steps information
  - `jq '.steps[]' manifest.json` : select only the steps
  - `jq '.steps[] | del(.resources)' manifest.json` :
  - `jq '.steps[] | del(.resources) | {name: .name, status: .status_completion}' manifest.json` : select specific fields
  - `jq '.steps[] | del(.resources) | select(.status_completion != "COMPLETED")' manifest.json` : select just the completed steps
  - `jq '.steps[] | .resources[] | .path_destination' manifest.json` : query the downloaded files
- Destroy the VM


# Run ETL

## Requirements

- Java:11
- Scala: 2.12
- gcloud cli

## Steps

- Clone the github repository [platform-etl-backend](http://github.com/opentargets/platform-etl-backend)
- Create and checkout a new branch for the release _release_23-12_
- Create the ETL configuration file named `configuration/<year>/<release>_platform.conf`. Make sure that the configuraiton file excludes the steps ot_crispr, encore and ot_crispr_validation (these steps are onli valid for ppp). Set the data_version, chembl_version and ensembl_version accordingly. This is an example of the configuration file:

```
spark-settings.write-mode = "ignore"

data_version = "23.12"
chembl_version = "33"
ensembl_version = "110"
evidences.data-sources-exclude = ["ot_crispr", "encore", "ot_crispr_validation"]
# update defaults for next release
etl-dag.resolve = false
```

- Copy the config file to the release bucket with the name platform.conf using the follow command: `gsutil cp configuration/<year>/<release>_platform.conf gs://open-targets-pre-data-releases/<release>/conf/`
- Build the JAR file for the ETL using`sbt -J-Xss2M -J-Xmx2G assembly`. The path for the JAR file will be `target/scala-<scala_version>/etl-backend-<commit>.jar`
- Copy the jar file to the release bucket using the follow command: `gsutil cp target/scala-<scala_version>/etl-backend-<commit>.jar gs://open-targets-pre-data-releases/<release>/jars/`- Create the Workflow configuration file with the name `configuration/<year>/workflow_<release>.conf`. Here you'll need to sepecify:

1. config.path: gs bucket path to the config file
2. config.file: name of the config file
3. jar.path: gs bucket path to the jar file
4. jar.file: name of the jar file

```
workflow-resources.config.path = "gs://open-targets-pre-data-releases/<release>/conf/"
workflow-resources.config.file = "platform.conf"
workflow-resources.jar.path = "gs://open-targets-pre-data-releases/<release>/jars/"
workflow-resources.jar.file = "etl-backend-<commit>.jar"
```

- Build the JAR file for the Workflow using`sbt -J-Xss2M -J-Xmx2G workflow/assembly`. The path for the JAR file will be `workflow/target/scala-<scala_version>/etl-backend-<commit>.jar`
- Run the workflow JAR to create a Dataproc cluster in which the ETL will run. To do this run the folowing command `java -jar <workflow_jar> workflow --config <workflow_config> public`. By using the `public` parameter we're running the etl for platform. You'll need to set:

1. workflow_jar: the path to the workflow JAR file
2. workflow_config: the path to the .conf file for workflow

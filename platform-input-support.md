#Platform Input Support

[Repo](https://github.com/opentargets/platform-input-support)


## Platform release

### Build
We commit the configuration for each release to the public repo. If there 
- *Clone* the repo
- *Checkout* a release branch from `master` e.g. `release_23-12`
- *Edit* the `./config.yaml`
    - Update versions for ***EFO***, ***Ensembl*** and ***CHEMBL*** - these versions are decided by the data team
        - **EFO**: one place to update
        - **CHEMBL**: several places to update
        - **Ensembl**: several places to update
- *Commit* and *push* to public repo - this triggers a [Travis build](https://app.travis-ci.com/github/opentargets/platform-input-support)
- *Wait* for the image to be published on [quay](https://quay.io/repository/opentargets/platform-input-support?tab=tags)

### Run PIS

#### Setup GCP infrastructure
*Note: this could be done by terraform*
- *Create* a [compute engine](https://console.cloud.google.com/compute/instancesAdd?cloudshell=false&project=open-targets-eu-dev) (vm) with the following specs
  - e2-standard-4
  - a boot disk with at least 500GB
  
#### Install dependencies
- *ssh* the machine `gcloud compute ssh --zone <ZONE> <VM_NAME> --project "open-targets-eu-dev"
`
- run the following (change the `IMAGE_TAG` and `RELEASE_VERSION` accordingly)
```bash
IMAGE_TAG="release_23-12"
RELEASE_VERSION="23.12"

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
sudo groupadd docker
sudo usermod -a -G docker $USER
newgrp docker
mkdir -p opentargets/credentials
mkdir -p opentargets/output
mkdir -p opentargets/log
gsutil cp gs://open-targets-ops/credentials/pis-service_account.json opentargets/credentials/open-targets-gac.json
```
#### Run the PIS container
```bash
tmux new -s pisrun
docker run -v /home/$USER/opentargets/output:/srv/output -v /home/$USER/opentargets/log:/usr/src/app/log -v /home/$USER/opentargets/credentials/open-targets-gac.json:/srv/credentials/open-targets-gac.json quay.io/opentargets/platform-input-support:$IMAGE_TAG -o /srv/output --log-level=DEBUG -gkey /srv/credentials/open-targets-gac.json -gb open-targets-pre-data-releases/$RELEASE_VERSION/input --exclude otar
```

#### Clean up
- Destroy the VM
---
icon: lucide/rocket
---

# Release Process

We have four releases per year (usually end of every Q): `YY.03`, `YY.06`, `YY.09`, `YY.12`.
We have two product favors per release: **Platform** and **PPP**.

## Platform


---

!!! note
    old docs below


## Platform Output Support

[Repo](https://github.com/opentargets/pos)

## Platform release
### Make the data images
- *Clone* the repo
- *Checkout* a release branch from `main` e.g. `release_23-12`
- *update* **release_ids**, **data_locations** and **datasource** in the profiles:
  - `./profiles/config.development`
- *Commit* and *push* to public repo
- Set the profile and create the data images
  ```bash
  make set_profile profile='development'
  make image
  ```
- The data images (one for clickhouse, one for elastic/opensearch) are built and then the POS VM is stops. Allow a few hours.
- Take a note of the details from the output e.g.
  ```python
  data_disk_images = {
    "clickhouse" = "posdevpf-20230921-0622-ch"
    "elastic_search" = "posdevpf-20230921-0622-es"
  }
  posvm = {
    "name" = "posdevpf-posvm-5tx89wx9"
    "username" = "otops"
    "zone" = "europe-west1-d"
  }
  ```
- The data disk images are needed for the terraform deployments
- The posvm details are needed for checking the progress (below)

### Check the progress
- From the output of the above, ssh the `posvm.name` as `posvm.username` e.g.
  ```bash
  gcloud compute ssh otops@posdevpf-posvm-tw97u257 --project open-targets-eu-dev
  ```
- The console will give you the google bucket where the logs are written e.g. `pos_gcp_path_pos_pipeline_session_logs = gs://open-targets-ops/logs/platform-pos/63ca794ff47d`. Save this for later for after the VM has been destroyed.
- To see the logs `pos_logs_startup`
- To follow the logs: `pos_logs_startup_tail`
- To check the end of the final POS logs in the google bucket from above for elastic/opensearch indices should all contain data except for `otar_projects` (except if ppp)
- Search the logs for "Count for" to check that the clickhouse tables all have records

### Release data
> [!WARNING]
> Only execute for production deployments
- Sync from pre-release to release bucket using the following: `make syncgs`
- Write data BigQuery
  - Write to DEV first and get checked by the team: `make bigquerydev`
  - Then if OK, write to PROD: `make bigqueryprod`
- EBI FTP release
  - Your EBI user must be a member of `opent` group.
  - Your EBI user must be able to become `otftpuser` virtual user.
  - Setup ssh keys with EBI cluster
    - Ensure you have a public key locally (e.g. using ssh-keygen)
    - Copy your public to the EBI cluster e.g. `ssh-copy-id -i  ~/.ssh/<KEY_NAME>.pub <USER>@codon-login`
  - Run the FTP sync script `make sync`

### Final steps
- Clean the infrastructure `make clean_image_infrastructure`



# Terraform Google Platform

- Clone the github repository [terraform-google-opentargets-platform](https://github.com/opentargets/terraform-google-opentargets-platform)

## Setup the profile
#### *PROD*
- Checkout a release branch from `main` e.g. `release_23-12`
- Get the necessary credentials for the deployment using `make credentials`
- Create a copy if the previous release profile e.g. `deployment_context.2312`. Update the necessary information for the release. This will usually include data versions, webapp version, api version, openai versions and any other changes specific to the release.
- Link the release profile to the production profile using `make update_linked_profile profile=2312 link_to_profile=production-platform`
- Set the current profile `make set_profile profile=production-platform`

#### *DEV*
- Set the current profile `make set_profile profile=dev-platform`

## Edit the profile and deploy
- Update the deployment profile with the new information.
- Evaluate the deployment plan by using `terraform plan` and verify that the output matches the expected changes
- Execute the plan using `terraform apply`

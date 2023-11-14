# Platform Output Support

[Repo](https://github.com/opentargets/platform-output-support)

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
- Pre-release `make syncgs`
- EBI FTP release `make sync`
- BigQuery `make bigqueryprod`

### Final steps
- Clean the infrastructure `make clean_image_infrastructure`

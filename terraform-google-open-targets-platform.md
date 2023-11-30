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

# Terraform Google Platform

- Clone the github repository [terraform-google-opentargets-platform](https://github.com/opentargets/terraform-google-opentargets-platform)
  
## Setup the profile
#### *PROD* 
- Checkout a release branch from `main` e.g. `release_23-12`
- Get the necessary credentials for the deployment using `make credentials`
- Create the deployment profile using `make clone_profile profile=<previous_release_profile> new_profile=<release_version>` e.g. `make clone_profile profile=2309 new_profile=2312`
- Link the release profile to the production profile using `make update_linked_profile profile=production-platform link_to_profile=2312`
- Set the current profile `make set_profile profile=production-platform`
  
#### *DEV*
- Set the current profile `make set_profile profile=dev-platform`
  
## Edit the profile and deploy
- Update the deployment profile with the new information.
- Evaluate the deployment plan by using `terraform plan` and verify that the output matches the expected changes
- Execute the plan using `terraform apply`

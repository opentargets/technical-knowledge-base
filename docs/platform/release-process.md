---
icon: lucide/rocket
---

# Release Process
* We have four releases per year, usually at the end of each quarter: `YY.03`,
  `YY.06`, `YY.09`, `YY.12`.
* There are two product flavors — **Platform** and **Partner Preview Platform
  (PPP)**. These run as coexisting deployments in separate namespaces. PPP is
  usually released one week after Platform. PPP follows the same mechanics in
  the `*-ppp` namespace with its own images and snapshots.

## Useful tools
- [GCP CLI](https://docs.cloud.google.com/sdk/docs/install-sdk#linux)
- [Terraform](https://developer.hashicorp.com/terraform/install)
- [Helm](https://helm.sh/docs/intro/install/), [Helm Diff Plugin](https://github.com/databus23/helm-diff#install)
- [k9s](https://k9scli.io/topics/install/)

## Overview
A release consist of:

1. Ideally a week before release date, start up staging with the new RC images and
  data snapshots.
2. Apply any fixes that are required there until everything is ready.
3. On release day, switch staging and production.

    !!! Note
        As staging is still up, if anything is wrong the rollback is instantaneous
        with `helm rollback` or switching back.

4. After a couple of days, bring down the old production.

This changes made editing [`profiles/production-platform.yaml`](https://github.com/opentargets/platform-deployment-nextgen/blob/main/profiles/production-platform.yaml).
And applied with `make deploy-chart-prod-platform`.


## Step-by-step
!!! Tip
    Check out the [config file for production](https://raw.githubusercontent.com/opentargets/platform-deployment-nextgen/refs/heads/main/profiles/production-platform.yaml).
    <br/>
    For this example, we assume `blue` is live and `green` is idle.

### 1. Update config
First, configure the idle colour with the upcoming release image tags and data
snapshots.

- In [the `green` block](https://github.com/opentargets/platform-deployment-nextgen/blob/main/profiles/production-platform.yaml#L29-L44),
    set the new `webapp`, `api` and `aiapi` images, the `clickhouse` / `opensearch`
    snapshots, and the `release` value.

    ```yaml title="profiles/production-platform.yaml" linenums="26" hl_lines="5 7 9 11 16 19"
        --8<-- "https://raw.githubusercontent.com/opentargets/platform-deployment-nextgen/refs/heads/main/profiles/profiles/production-platform.yaml:26:47"
    ```

- At [the start of the file](https://github.com/opentargets/platform-deployment-nextgen/blob/main/profiles/production-platform.yaml#L7-L8),
    Set `staging: enabled` so the staging deployment is started.

    ```yaml title="profiles/production-platform.yaml" linenums="1" hl_lines="8"
        --8<-- "https://raw.githubusercontent.com/opentargets/platform-deployment-nextgen/refs/heads/main/profiles/production-platform.yaml:1:10"
    ```

!!! Note
    During testing, `-rc`/`-dev` image tags and `-rcN` snapshots are expected here.

### 2. Bring up staging
```bash
make deploy-chart-prod-platform
```

Review the `helm diff` and confirm. You should see a lot of additions (green lines)
as Helm brings up another whole copy of the infra. Green starts on the `staging`
node pool (which scales up from 0) and is served at `staging.platform.opentargets.org`
(plus `api.` / `ai.`).

### 3. Test
First checks you can do:

- Use k9s to check that pods are starting properly. Type `:namespaces`, select
    `production-platform` and check the `production-platform-green-*` pods by
    scrolling to them and pressing `d`.
- Confirm the API reports the expected release and version. For that, go to the
    [Staging API GraphiQL](https://api.staging.platform.opentargets.org/api/v4/graphql/browser)
    and run the following query:

    ```graphql title="API Version query"
        {
            meta {
                product
                dataPrefix
                apiVersion {
                    x
                    y
                    z
                }
                dataVersion {
                    year
                    month
                    iteration
                }
            }
        }
    ```

- Head into [the staging platform](https://staging.platform.opentargets.org) and
    check that things look as they should.

### 4. Promote to production
On release day, and once all tests are confirmed to be passing and things look
correct:

- **Swap every `-rc` / `-dev` image tag and `-rcN` snapshot in the `green` block
    to its final value** (see [Final versions](#final-versions)).
- Set `production: green`.
- Re-apply: `make deploy-chart-prod-platform`. The prod Services now select green
    pods. Blue keeps running (now the staging colour) as a rollback.

### 5. Decommission staging
Once the release is confirmed healthy and has been in production for a few days:

- Set `staging: disabled`.
- Re-apply: `make deploy-chart-prod-platform`. Blue's workloads and their PVCs are
    removed and the `staging` node pool scales back to 0.

### Rollback
While the old colour is still up (before step 5), rollback is immediate: set
`production` back to the previous colour and re-apply. After decommission, you
must bring the old colour back up as staging first, then promote it.

## Final versions
Staging runs release-candidate artifacts; production must run final ones. As the
time to release approaches, appropriate tags should be created for [ot-ui-apps](https://github.com/opentargets/ot-ui-apps),
[platform-api](https://github.com/opentargets/platform-api) and [ot-ai-api](https://github.com/opentargets/ot-ai-api).
Same with the ClickHouse and OpenSearch Google Cloud Snapshots.

- **Image tags:** replace `…-rc.N` with the final release tag (e.g. `26.03.1-rc.4` → `26.03.1`).
- **Snapshots:** replace `platform-2603-ch-rcN` with the final `platform-2603-ch` (same for `-os`).

!!! warning
    **Final versions should be running in staging before release day!**
    <br/>
    Changing a ClickHouse or OpenSearch snapshot recreates its StatefulSet — the
    snapshot name is part of the StatefulSet name — and triggers a full reload
    from the new snapshot. It is not an in-place update, so be sure to do this
    before the actual release.

## Checklists

### Pre-release
- [ ] [ot-ui-apps](https://github.com/opentargets/ot-ui-apps/tags) tag present
    and has run its CI/CD; image tag exists in [Artifact Registry](https://console.cloud.google.com/artifacts/docker/open-targets-eu-dev/europe-west1/ot-ui-apps/ot-ui-apps?project=open-targets-eu-dev).
- [ ] [platform-api](https://github.com/opentargets/platform-api/tags) tag present
    and has run its CI/CD; image tag exists in [Artifact Registry](https://console.cloud.google.com/artifacts/docker/open-targets-eu-dev/europe-west1/platform-api/platform-api?project=open-targets-eu-dev).
- [ ] [ot-ai-api](https://github.com/opentargets/ot-ai-api/tags) tag present and
    has run its CI/CD; image tag exists in [Artifact Registry](https://console.cloud.google.com/artifacts/docker/open-targets-eu-dev/europe-west1/ot-ai-api/ot-ai-api?project=open-targets-eu-dev).
- [ ] ClickHouse and OpenSearch release snapshots created in `open-targets-eu-dev`.
- [ ] Idle colour configured and `staging: enabled`.

### Promotion
- [ ] All `-rc` / `-dev` image tags swapped to final.
- [ ] All `-rcN` snapshots swapped to final.
- [ ] `production` flipped to the new colour.

### Post-release
- [ ] Create the release branch/tag in [`ot-snapshot`](https://github.com/opentargets/ot-snapshot),
    pinning the final component versions.
- [ ] Staging decommissioned (`staging: disabled`) after the grace period.
- [ ] Production re-verified after teardown of staging.

## Troubleshooting
!!! Tip
    We should write down usual problems and the solution here.

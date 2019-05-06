# Notes for Internal Contributors

The notes here are primarily targeted at internal (CircleCI) contributors to the orb but could be of reference to fork owners who wish to run the tests with their own Azure account.

## Building

### Required Project Environment Variables

The following [project environment variables](https://circleci.com/docs/2.0/env-vars/#setting-an-environment-variable-in-a-project) must be set for the project on CircleCI via the project settings page, before the project can be built successfully.

| Variable                       | Description                      |
| -------------------------------| ---------------------------------|
| `AZURE_USERNAME`            | Azure user account name              |
| `AZURE_PASSWORD`        | Azure user account password              |
| `RESOURCE_NAME_PREFIX`     | Prefix for some resources created in tests. This is used just to make the project more portable.                |
| `CIRCLECI_API_KEY`             | Used by the `queue` orb          |

### Required Context and Context Environment Variables

The `orb-publishing` context is referenced in the build. In particular, the following [context environment variables](https://circleci.com/docs/2.0/env-vars/#setting-an-environment-variable-in-a-context) must be set in the `orb-publishing` context, before the project can be built successfully.

| Variable                       | Description                      |
| -------------------------------| ---------------------------------|
| `CIRCLE_TOKEN`                 | CircleCI API token used to publish the orb  |

### SSH configuration

### AKS Resource Cleanup

Here are some useful commands for manual deletion of resources in case of any lingering resources:

#### Resource Group

Deleting a resource group is the most convenient way to delete a cluster and its associated resources. However, the Service Principal for the cluster must still be deleted in a separate step.

```
az group delete --name <resource-group-name> --yes
```

#### Service Principal

##### Delete all Service Principals

This assumes the cluster names all contain "orb"

```
az ad sp list | jq -r '.[] | select (.displayName | contains("orb")) | .objectId' | xargs -n1 az ad sp delete --id
```

##### Delete the Service Principal for a cluster

```
az ad sp list | jq -r '.[] | select (.displayName | contains("CLUSTERNAME")) | .objectId' | xargs -n1 az ad sp delete --id
```

## Orb Publishing
The orb is promoted into publishing by triggering the `production-orb-publishing` workflow. This workflow is meant to be manually triggered by tagging a `master` branch commit, after (manually) verifying that it has successfully completed the `integration-tests` workflow. The `production-orb-publishing` workflow will promote into production a dev orb that was published with a version that corresponds to the shortened SHA-1 hash of the tagged commit. The workflow has a manual approval step before it can proceed to promotion of the orb.

### Patch release tag example:

```
git tag patch-release-v0.0.1
git push origin patch-release-v0.0.1
```

### Minor release tag example:

```
git tag minor-release-v0.1.0
git push origin minor-release-v0.1.0
```

### Major release tag example:

```
git tag major-release-v1.0.0
git push origin major-release-v1.0.0
```

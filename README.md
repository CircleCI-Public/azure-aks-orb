# Azure AKS Orb [![CircleCI status](https://circleci.com/gh/CircleCI-Public/azure-aks-orb.svg "CircleCI status")](https://circleci.com/gh/CircleCI-Public/azure-aks-orb) [![CircleCI Orb Version](https://img.shields.io/badge/endpoint.svg?url=https://badges.circleci.io/orb/circleci/azure-aks)](https://circleci.com/orbs/registry/orb/circleci/azure-aks) [![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/circleci-public/azure-aks-orb/master/LICENSE) [![CircleCI Community](https://img.shields.io/badge/community-CircleCI%20Discuss-343434.svg)](https://discuss.circleci.com/c/ecosystem/orbs)

A CircleCI Orb to simplify deployments to Azure Kubernetes Service (AKS).

Here are some features that the Azure AKS orb provides:

- Setting up and tearing down of AKS clusters via the `create-cluster` and `delete-cluster` commands / jobs
- Allowing `kubectl` and other tools that access `kubeconfig` (like `helm`) to work with an AKS cluster via the `update-kubeconfig-with-credentials` command
- Updating deployments with container image updates via the `update-container-image` job
- Installing helm and helm charts (See: `install-helm-on-cluster` and `install-helm-chart`)

## Usage

See the [orb registry listing](http://circleci.com/orbs/registry/orb/circleci/azure-aks) for usage guidelines.

## Requirements

- `curl` should be present in `PATH`.

- A Python environment that supports usage of the Azure CLI.

## Examples

Full usage examples can be found on the Azure AKS orb's page in the orb registry, [here](https://circleci.com/orbs/registry/orb/circleci/azure-aks#usage-examples).

```
version: 2.1

orbs:
  azure-aks: circleci/azure-aks@0.1.1
  kubernetes: circleci/kubernetes@0.4.0

jobs:
  create-deployment:
    executor: azure-aks/default
    parameters:
      cluster-name:
        description: |
          Name of the AKS cluster
        type: string
      resource-group:
        description: |
          Resource group that the cluster is in
        type: string
    steps:
      - checkout
      - azure-aks/update-kubeconfig-with-credentials:
          cluster-name: << parameters.cluster-name >>
          resource-group: << parameters.resource-group >>
          install-kubectl: true
          perform-login: true
      - kubernetes/create-or-update-resource:
          resource-file-path: "tests/nginx-deployment/deployment.yaml"
          resource-name: "deployment/nginx-deployment"

workflows:
  deployment:
    jobs:
      - azure-aks/create-cluster:
          cluster-name: aks-demo-deployment
          resource-group: aks-demo-deployment-rg
          create-resource-group: true
          location: eastus
          generate-ssh-keys: true
      - create-deployment:
          cluster-name: aks-demo-deployment
          resource-group: aks-demo-deployment-rg
          requires:
            - azure-aks/create-cluster
      - azure-aks/update-container-image:
          cluster-name: aks-demo-deployment
          resource-group: aks-demo-deployment-rg
          resource-name: "deployment/nginx-deployment"
          container-image-updates: "nginx=nginx:1.9.1"
          record: true
          requires:
            - create-deployment
          post-steps:
            - kubernetes/delete-resource:
                resource-types: "deployment"
                resource-names: "nginx-deployment"
                wait: true
      - azure-aks/delete-cluster:
          cluster-name: aks-demo-deployment
          resource-group: aks-demo-deployment-rg
          delete-service-principal: true
          delete-resource-group: true
          requires:
            - azure-aks/update-container-image
```


## Contributing

We welcome [issues](https://github.com/CircleCI-Public/azure-aks-orb/issues) to and [pull requests](https://github.com/CircleCI-Public/azure-aks-orb/pulls) against this repository!

For further questions/comments about this or other orbs, visit [CircleCI's orbs discussion forum](https://discuss.circleci.com/c/orbs).

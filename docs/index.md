# GitHub workflow Kubernetes validation

> [!IMPORTANT]
>
> This workflow depends on the deprecated engineering-docker-images and will be
> removed together with them. Use the Kubernetes and pallet workflows in
> [coopnorge/mage](https://github.com/coopnorge/mage) instead:
> `reusable-pallets.yaml` for pallet validation and `reusable-kubernetes.yaml`
> (called through `goapp.yaml`) for Kubernetes validation.
>
> The mage `pallets`, `catalog-info` and `policy-bot` targets cover repos that
> only validate `.pallet` files and catalog-info. Helm, Kustomize and Kyverno
> validation for every repo type is not covered yet
> ([mage#924](https://github.com/coopnorge/mage/issues/924)), nor is pallet
> validation for every bundle
> ([mage#925](https://github.com/coopnorge/mage/issues/925)). The Argo CD diff
> has no replacement yet. Progress is tracked in
> [mage#923](https://github.com/coopnorge/mage/issues/923) and
> [engineering-docker-images#4456](https://github.com/coopnorge/engineering-docker-images/issues/4456).

## Goal

> `v1` documentation below

Validation of Kubernetes templates and also a rich diff to state in cluster.

## Usage

The workflow uses the Kubernetes engineering image to run the CI steps. The
image auto-detects Kubernetes objects required validating and can also produce a
diff in case of changes. To use this in CI, set up the workflow like this:

```yaml
kubernetes-ci:
  name: "Kubernetes CI"
  needs: ["setup"]
  permissions:
    contents: read
    pull-requests: write
  concurrency:
    group:
      ${{ github.repository }}-${{ github.workflow }}-kubernetes-ci-${{
      github.ref }}
    cancel-in-progress: true
  if: ${{ needs.setup.outputs.run-kubernetes-ci == 'true'}}
  uses: coopnorge/github-workflow-kubernetes-validation/.github/workflows/kubernetes-validation.yaml@v2.0.0
  secrets:
    argocd-api-token: ${{ secrets.ARGOCD_API_TOKEN }}
```

And have in your `docker-compose.yaml`

```yaml
kubernetes-devtools:
  build:
    context: devtools
    target: kubernetes-devtools
    dockerfile: Dockerfile
  privileged: false
  command: validate
  security_opt:
    - seccomp:unconfined
    - apparmor:unconfined
  volumes:
    - .:/srv/workspace:z
    - $HOME/.argocd:/root/.config/argocd
```

and in `devtools/Dockerfile` have at least this image

```dockerfile
FROM ghcr.io/coopnorge/engineering-docker-images/e0/devtools-kubernetes-v1beta1:latest@sha256:6cab3cd24ce510d11105deb0777df7d2c2e959eaed44e049d5ecd2304e217a12 AS kubernetes-devtools
```

Make sure you update your Dependabot config to update docker images in
`devtools/Dockerfile`

## Inputs

```yaml
inputs:
  do-argocd-diff:
    required: false
    default: true
    type: boolean
    description: |
      Boolean to enable argocd diff.
  docker-compose-file:
    type: string
    default: ./docker-compose.yaml
    required: false
    description: |
      Location of the docker-compose file relative to the root of the repository
  docker-compose-service:
    type: string
    default: kubernetes-devtools
    required: false
    description: |
      The name of the service in the docker-compose file to use for validating terraform
  argocd-server:
    type: string
    default: argocd.internal.coop
    required: false
    description: |
      The dns server of ArgoCD. Defaults to 'argocd.internal.coop'
  validate-target:
    type: string
    default: validate
    required: false
    description: |
      Target to run when running the validate step, defaults to 'validate'

secrets:
  argocd-api-token:
    required: false
    description: |
      Token to get read only access to argocd. This is a global secret secrets.ARGOCD_API_TOKEN
```

## V1 Docs

### v1 Inputs

```yaml
inputs:
  environment-matrix-json:
    required: true
    type: string
    description: |
      A json array containing info about environments. Required keys are 'destination-cluster'. That can be api-production
      or api-staging. If do-argocd-diff=true then 'argocd-appname' and 'argocd-tracking-directory' are also required.
      Example:
        [{
            "argocd-appname": "helloworld-api-production-helloworld",
            "argocd-tracking-directory": "kubernetes/kustomize-deployments/production" ,
            "destination-cluster" : "api-production",
          },
          {
            "argocd-appname": "helloworld-api-staging-helloworld",
            "argocd-tracking-directory": "kubernetes/kustomize-deployments/staging" ,
            "destination-cluster" : "api-staging",
        }]
    do-argocd-diff:
      required: false
      default: true
      type: boolean
      description: |
        Boolean to enable argocd diff.
    manifest-generator:
      required: false
      default: kustomize
      type: string
      description: |
        Which tool use to render manifests. Currently supported: kustomize (default), helm and plain
  secrets:
    argocd-api-token:
      required: false
      description: |
        Token to get read only access to argocd. This is a global secret secrets.ARGOCD_API_TOKEN
```

### Example

```yaml
jobs:
  <some other jobs jobs>
  kubernetes-ci:
    name: "Kubernetes CI"
    uses: coopnorge/github-workflow-kubernetes-validation/.github/workflows/kubernetes-validation.yaml@v1
    with:
      environment-matrix-json: >-
        [{
            "argocd-appname": "helloworld-api-production-helloworld",
            "argocd-tracking-directory": "kubernetes/kustomize-deployments/production" ,
            "destination-cluster" : "api-production",
          },
          {
            "argocd-appname": "helloworld-api-staging-helloworld",
            "argocd-tracking-directory": "kubernetes/kustomize-deployments/staging" ,
            "destination-cluster" : "api-staging",
        }]
    secrets:
      argocd-api-token: ${{ secrets.ARGOCD_API_TOKEN }}
```

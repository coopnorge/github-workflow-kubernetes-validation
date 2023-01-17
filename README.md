# github-workflow-kubernetes-validation


## Goal

Validation of kubernetes templates and also a rich diff to state in cluster.

## Inputs

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

## Example

```yaml
jobs: 
  <some other jobs jobs>
  kubernetes-ci:
    name: "Kubernetes CI "
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

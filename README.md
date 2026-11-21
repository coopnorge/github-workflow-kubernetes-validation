# Kubernetes Validation Workflow for GHA

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

[docs](docs/index.md)

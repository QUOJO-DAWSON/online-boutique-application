# Warning Reference Only

The manifests in this directory are **not** the source of truth for deployments.

All production deployments are managed via GitOps in the
[online-boutique-gitops](https://github.com/QUOJO-DAWSON/online-boutique-gitops) repository.

The `productcatalogservice/deploy.yaml` file in this directory is updated automatically
by the CI pipeline to track the latest image tag - it serves as a record of the last
built image, not a deployment target.

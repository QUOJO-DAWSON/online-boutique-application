# online-boutique-application

> Source code, Dockerfiles, and CI pipelines for the Online Boutique microservices application. Images are built, scanned, and pushed to Docker Hub — then deployed to EKS via GitOps in [online-boutique-gitops](https://github.com/gdawsonkesson/online-boutique-gitops), running on the platform provisioned in [eks-infra-automation](https://github.com/gdawsonkesson/eks-infra-automation).

---

## What This Is

This is the application layer of a three-repo DevOps portfolio. It owns the source code and the build pipeline — nothing else. It does not define how the app is deployed, what cluster it runs on, or what policies govern it. Those concerns live in their own repos.

The separation is intentional. In a real engineering org, the team that writes the code doesn't own the deployment manifests, and the platform team doesn't own the application source. This repo reflects that boundary.

---

## Three-Repo Architecture
```
online-boutique-application  (this repo)
        │
        │  builds images → pushes to Docker Hub
        │  triggers repository_dispatch
        ▼
online-boutique-gitops
        │
        │  Kustomize overlays updated with new image tags
        │  ArgoCD detects change, syncs to cluster
        ▼
eks-infra-automation
        │
        │  EKS cluster, ArgoCD, Kyverno, Istio, Prometheus
        └─ Platform that runs everything
```

---

## Services

| Service | Language | Responsibility |
|---------|----------|---------------|
| adservice | Java | Serves context-based ads |
| cartservice | C# | Manages shopping cart via Redis |
| checkoutservice | Go | Orchestrates the checkout flow |
| currencyservice | Node.js | Currency conversion using ECB rates |
| emailservice | Python | Sends order confirmation emails |
| frontend | Go | Serves the web UI, aggregates backend calls |
| paymentservice | Node.js | Processes payments via Stripe |
| productcatalogservice | Go | Serves product listings from JSON |
| recommendationservice | Python | Returns product recommendations |
| shippingservice | Go | Calculates shipping costs |
| redis-cart | Redis | Cart data store |

---

## CI/CD Pipeline

Two workflows handle the build pipeline:

### `docker/dockerbuild-microservices.yaml` — Bulk build (push trigger)
Builds and pushes Docker images for all services except productcatalogservice on every push to main. No tests, no quality gates — pure build and push. Services covered: adservice, cartservice, checkoutservice, currencyservice, emailservice, frontend, paymentservice, recommendationservice, shippingservice.

### `.github/workflows/CI_productcatalogue.yaml` — Dedicated productcatalog pipeline (PR trigger)
productcatalogservice has its own dedicated pipeline because it is the only service with Go unit tests and lint checks. The pipeline runs in sequence:
```
Build & Test → Code Quality (golangci-lint) → Docker Build + Trivy Scan → Update K8s manifest → Trigger GitOps
```

The final step fires a `repository_dispatch` to `online-boutique-gitops`, which automatically updates the `overlays/dev/kustomization.yaml` image tag via a `yq` patch — no manual intervention required.

---

## Image Tagging Convention

All images follow the convention `dawsonkesson/<service>:v<github_run_id>` — for example `dawsonkesson/frontend:v16319814565`. No `latest` tags are ever pushed. This is enforced by the Kyverno `disallow-latest-tag` policy on the EKS cluster, which will block any deployment that attempts to use a mutable tag.

---

## Security

**Trivy container scanning** runs in the productcatalog pipeline after the Docker build. The image tarball is scanned for CRITICAL and HIGH CVEs before being pushed. Results are uploaded to the GitHub Security tab as SARIF. The scan runs with `exit-code: 0` — it reports but does not block on vulnerabilities in upstream dependencies outside our control.

**No secrets in code or CI environment.** The Stripe API key is pulled from AWS Secrets Manager at runtime by the External Secrets Operator running in the cluster. The only secrets in GitHub Actions are Docker Hub credentials and a PAT for cross-repo dispatch.

---

## Repository Structure
```
online-boutique-application/
├── .github/
│   └── workflows/
│       └── CI_productcatalogue.yaml   # Full CI pipeline for productcatalogservice
├── docker/
│   └── dockerbuild-microservices.yaml # Bulk image build for all other services
├── kubernetes/                         # ⚠️ Reference only — see note below
│   ├── README.md
│   └── <service>/deploy.yaml          # Last-built image tag record per service
└── src/
    ├── adservice/
    ├── cartservice/
    ├── checkoutservice/
    ├── currencyservice/
    ├── emailservice/
    ├── frontend/
    ├── paymentservice/
    ├── productcatalogservice/
    ├── recommendationservice/
    ├── shippingservice/
    └── shoppingassistantservice/
```

> The `kubernetes/` directory is **not** the deployment source of truth. All deployments are managed via [online-boutique-gitops](https://github.com/gdawsonkesson/online-boutique-gitops). The `productcatalogservice/deploy.yaml` in this folder is updated automatically by CI as a tag tracking record only.

---

## Local Development

Run productcatalogservice locally:
```bash
cd src/productcatalogservice
go mod download
go build -o product-catalog-service
./product-catalog-service
```

For full application deployment, the cluster and GitOps setup is defined in the platform and gitops repos.

---

## Required GitHub Secrets

| Secret | Purpose |
|--------|---------|
| `DOCKERHUB_USERNAME` | Docker Hub login |
| `DOCKERHUB_TOKEN` | Docker Hub push access |
| `ACTIONS_PA_TOKEN` | Cross-repo dispatch to online-boutique-gitops |
| `USER_EMAIL` | Git commit identity for manifest updates |
| `USER_NAME` | Git commit identity for manifest updates |

---

## Upstream

This project is built on top of [Google Cloud Platform's microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo), modified for this portfolio with custom CI pipelines, Docker Hub publishing, and GitOps integration.

---

## Author

**George Dawson-Kesson** — AWS Certified Solutions Architect – Associate (SAA-C03)
Portfolio: [gdawsonkesson.com](https://gdawsonkesson.com)
GitHub: [gdawsonkesson](https://github.com/gdawsonkesson)
Platform repo: [eks-infra-automation](https://github.com/gdawsonkesson/eks-infra-automation)
GitOps repo: [online-boutique-gitops](https://github.com/gdawsonkesson/online-boutique-gitops)

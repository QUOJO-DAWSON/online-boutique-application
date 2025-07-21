# Online Boutique Application

This project is a full DevOps pipeline built around a cloud-native microservices-based application inspired by [Google's Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo). It demonstrates modern best practices in CI/CD, Kubernetes, observability, GitOps, and infrastructure as code (IaC).

## Why This Project?

This project was built to:

- Showcase end-to-end DevOps expertise and best practices
- Demonstrate practical implementation of CI/CD pipelines
- Provide a real-world example of containerized microservices
- Illustrate security integration in the deployment pipeline
- Function as a portfolio piece for DevOps engineers

## Overview

This repository contains the source code and deployment configurations for the Online Boutique application. The application is a cloud-native microservices demo that simulates an e-commerce platform. It showcases how multiple microservices can work together in a Kubernetes environment with proper CI/CD practices.

## Features

- **Kubernetes-native microservices architecture**
- **GitHub Actions for Automated CI/CD pipelines**
- **GitOps-style continuous delivery**
- **Container security scanning with Trivy**
- **Microservices communication via gRPC**
- **Containerized deployment with Docker**
- **Scalable Kubernetes deployments** 

## Tools and Technologies

- **Containerization**: Docker, Kubernetes
- **CI/CD**: GitHub Actions
- **Security**: Trivy for container scanning
- **Deployment**: GitOps with separate configuration repository

## Architecture

The application consists of the following microservices:

- **Frontend**: Web interface
- **ProductCatalogService**: Product catalog API
- **CartService**: Shopping cart service
- **CheckoutService**: Checkout processing
- **CurrencyService**: Currency conversion
- **EmailService**: Email notification service
- **PaymentService**: Payment processing
- **RecommendationService**: Product recommendation
- **ShippingService**: Shipping cost calculation
- **AdService**: Advertisements service
- **Redis-Cart**: Redis database for cart data

## CI/CD Pipeline

The repository includes GitHub Actions workflows for continuous integration and deployment:

- **Build**: Compiles and tests the code
- **Code-Quality**: Runs linting and code quality checks
- **Docker**: Builds and pushes Docker images with security scanning
- **Update Kubernetes Manifests**: Updates deployment files with new image tags
- **GitOps Trigger**: Triggers deployment in the GitOps repository

### GitHub Actions Secrets

The following secrets need to be configured in your GitHub repository:

- `DOCKERHUB_USERNAME`: Your Docker Hub username
- `DOCKERHUB_TOKEN`: Your Docker Hub access token
- `ACTIONS_PA_TOKEN`: GitHub Personal Access Token with repo scope
- `USER_EMAIL`: Email for git commits
- `USER_NAME`: Username for git commits

## Development

### Prerequisites

- Go 1.22+
- Docker
- Kubernetes cluster
- GitHub account with repository access

### Local Development

1. Clone the repository:
   ```
   git clone https://github.com/your-username/online-boutique-application.git
   cd online-boutique-application
   ```

2. Run a microservice locally (example with ProductCatalogService):
   ```
   cd src/productcatalogservice
   go mod download
   go build -o product-catalog-service
   ./product-catalog-service
   ```

3. For full application deployment, use the Kubernetes manifests in the `kubernetes/` directory.

## Deployment

The application is deployed using a GitOps approach:

1. Changes are pushed to this repository via pull request
2. GitHub Actions CI pipeline builds and tests the code
3. Docker images are built, scanned for vulnerabilities with Trivy, and pushed to Docker Hub
4. Kubernetes manifests are automatically updated with new image tags
5. A separate GitOps repository is triggered to deploy the changes to the Kubernetes cluster

## Security

- Container images are scanned for vulnerabilities using Trivy during the CI process
- Scan results are uploaded to GitHub Security tab for visibility and tracking
- Critical vulnerabilities can block the deployment pipeline
- All microservices run with appropriate resource limits and non-root users

## License

This project is available under the Apache 2.0 license. 
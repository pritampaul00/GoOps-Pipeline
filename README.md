# GoOps Pipeline

[![Go](https://img.shields.io/badge/Go-1.22.5-00ADD8?logo=go&logoColor=white)](https://go.dev/)
[![Docker](https://img.shields.io/badge/Container-Docker-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Kubernetes](https://img.shields.io/badge/Orchestration-Kubernetes-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Helm](https://img.shields.io/badge/Packaging-Helm-0F1689?logo=helm&logoColor=white)](https://helm.sh/)
[![GitOps](https://img.shields.io/badge/GitOps-Argo_CD-EF7B4D)](https://argo-cd.readthedocs.io/)

A Go web application delivered through a cloud-native DevOps workflow. This repository brings together Docker, Kubernetes, Helm, GitHub Actions, Argo CD, and Amazon EKS to demonstrate automated build, image publishing, and GitOps-based deployment.

## What it does

The application serves a small static website through Go's standard `net/http` package.

| Route | Page |
|---|---|
| `/home` | Home |
| `/courses` | Courses |
| `/about` | About |
| `/contact` | Contact |

The delivery workflow is designed to:

- Build and test the Go application with GitHub Actions.
- Build a multi-stage Docker image with a Distroless runtime.
- Publish the image to Docker Hub.
- Update the image tag in the Helm values file.
- Let Argo CD detect the Git change and synchronize the Kubernetes deployment.

## Architecture

```text
Developer push to main
        |
        v
  GitHub Actions
  | build and test
  | lint
  v
  Docker image build
        |
        v
     Docker Hub
        |
        v
 Update Helm image tag
        |
        v
  Git repository change
        |
        v
      Argo CD
        |
        v
   Amazon EKS
        |
        v
 NGINX Ingress -> Go web app
```

The repository includes application code, container configuration, Kubernetes manifests, a Helm chart, and installation notes for NGINX Ingress and Argo CD. AWS cluster provisioning and Argo CD Application configuration are not automated by the files currently included.

## Tech stack

| Area | Technology |
|---|---|
| Application | Go 1.22.5, `net/http` |
| Web content | Static HTML |
| Container build | Docker multi-stage build |
| Runtime image | Distroless |
| Registry | Docker Hub |
| Orchestration | Kubernetes, Amazon EKS |
| Packaging | Helm |
| Ingress | NGINX Ingress Controller |
| CI | GitHub Actions |
| GitOps CD | Argo CD |

## Repository layout

```text
.
├── .github/workflows/cicd.yaml  # Build, test, lint, publish, update chart tag
├── gitops/argocd/               # Argo CD installation guide
├── helm/go-web-app-chart/       # Helm chart and templates
├── ingress-controller/nginx/    # NGINX Ingress installation guide
├── k8s/manifests/               # Raw Kubernetes manifests
├── static/                      # HTML pages and image assets
├── Dockerfile
├── go.mod
├── main.go
└── main_test.go
```

## Requirements

For local development:

- Go 1.22.5 or a compatible Go 1.22 release.

For container builds:

- Docker.

For Kubernetes deployment:

- A reachable Kubernetes cluster.
- `kubectl` and Helm.
- An installed NGINX Ingress Controller if you need external ingress.
- For the documented cloud path: AWS CLI, an Amazon EKS cluster, and Argo CD.

## Quickstart: run locally

Clone the repository and enter its directory:

```bash
git clone <repository-url>
cd <repository-directory>
```

Run the application:

```bash
go run .
```

Open one of the routes:

```text
http://localhost:8080/home
http://localhost:8080/courses
http://localhost:8080/about
http://localhost:8080/contact
```

The server listens on `0.0.0.0:8080`. Stop it with `Ctrl+C`.

## Build and test

Build the Go binary:

```bash
go build -o go-web-app .
```

Run tests:

```bash
go test ./...
```

Run tests with the race detector:

```bash
go test -race ./...
```

## Build and run with Docker

Build the image:

```bash
docker build -t go-web-app:local .
```

Run the container:

```bash
docker run --rm -p 8080:8080 go-web-app:local
```

Then visit `http://localhost:8080/courses`.

To publish an image, tag it for your Docker Hub account and push it:

```bash
docker tag go-web-app:local <dockerhub-username>/go-web-app:<tag>
docker push <dockerhub-username>/go-web-app:<tag>
```

## Deploy to Kubernetes

The repository contains raw manifests in `k8s/manifests/`. Update the image in `deployment.yaml` to a tag that exists in your registry, then apply the resources:

```bash
kubectl apply -f k8s/manifests/
kubectl get deployments,pods,services,ingress
```

The Service exposes port `80` and forwards traffic to container port `8080`.

## Deploy with Helm

The chart is located at `helm/go-web-app-chart/`. Its current default image repository is `pritam954/go-web-app`, and its image tag is set in `values.yaml`.

Install or upgrade the release:

```bash
helm upgrade --install go-web-app ./helm/go-web-app-chart
```

Inspect the rendered resources before deploying:

```bash
helm template go-web-app ./helm/go-web-app-chart
```

The chart currently hardcodes some values in its templates. See [Known issues](#known-issues-and-follow-up-work) before relying on overrides such as `image.repository`, `replicaCount`, or `ingress.enabled`.

## CI/CD and GitOps

The workflow at `.github/workflows/cicd.yaml` runs on pushes to `main`, except for changes limited to the paths excluded by its trigger. It builds and tests the Go application, runs golangci-lint, builds and pushes a Docker image, then updates the image tag in the Helm values file.

Configure these GitHub Actions repository secrets before enabling image publishing:

| Secret | Purpose |
|---|---|
| `DOCKERHUB_USERNAME` | Docker Hub account used for publishing |
| `DOCKERHUB_TOKEN` | Docker Hub access token |
| `TOKEN` | GitHub token used by the tag-update job to push the chart change |

Argo CD must already be installed and configured to watch this repository and the chart path. The repository provides an installation walkthrough at [`gitops/argocd/installation.md`](gitops/argocd/installation.md). The NGINX controller walkthrough is at [`ingress-controller/nginx/installation.md`](ingress-controller/nginx/installation.md).

For an EKS deployment, first configure AWS credentials and `kubectl` access to the cluster. The repository documents cluster usage and controller installation, but does not include infrastructure-as-code for provisioning EKS.

## Demo

Run locally and open:

- [Home](http://localhost:8080/home)
- [Courses](http://localhost:8080/courses)
- [About](http://localhost:8080/about)
- [Contact](http://localhost:8080/contact)

A screenshot of the application is available at [`static/images/golang-website.png`](static/images/golang-website.png).

## Contributing

Contributions are welcome.

1. Fork the repository and create a focused branch.
2. Make your change and add or update tests.
3. Run `gofmt -w *.go` and `go test ./...`.
4. For deployment changes, validate manifests with `helm lint helm/go-web-app-chart/` and inspect output with `helm template`.
5. Open a pull request with a clear summary, rationale, and test evidence.

Keep changes small and document any new configuration or operational steps. Be respectful and constructive in reviews. Harassment, discrimination, and personal attacks are not acceptable. This project does not currently include a separate formal Code of Conduct; contributors should follow these expectations until one is adopted.

## Known issues and follow-up work

The following items are based on the repository contents at review time:

- The Helm Deployment template hardcodes the image repository and replica count instead of using `.Values.image.repository` and `.Values.replicaCount`.
- The Helm Ingress template is rendered unconditionally, even though `ingress.enabled` defaults to `false`. Its host and ingress class are also hardcoded.
- The workflow's lint job is separate from the image-publishing job's dependencies. The publish job waits for `build`, but not for `code-quality`.
- The workflow uses `actions/setup-go@v2` with Go `1.22`, while `go.mod` specifies `1.22.5`. Pin and align the action and Go version.
- The existing test covers only the `/home` handler's status and content type. Add coverage for all routes and missing paths.
- The raw Kubernetes manifests and Helm chart duplicate resource definitions. Choose one deployment source of truth where practical.
- The repository contains a file named `main` at its root. Confirm whether it is an intentional tracked binary and remove it from version control if it is a generated build artifact.
- The Ingress rewrite annotation and path behavior should be verified against the intended URL routing.

Suggested roadmap:

1. Make Helm templates fully values-driven and honor `ingress.enabled`.
2. Expand route tests and add CI coverage reporting.
3. Make lint a required dependency before publishing images.
4. Pin workflow action versions and add dependency/security checks.
5. Add readiness and liveness probes, resource requests and limits, and a non-root runtime user.
6. Add an example Argo CD Application manifest and document DNS/TLS configuration.
7. Add a formal Code of Conduct and a release/versioning policy.

## License

No license file was present in the supplied repository. Add a `LICENSE` file before distributing the project under an open-source license.

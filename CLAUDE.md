# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Kustomize overlay that deploys upstream Argo CD into a Kubernetes cluster (namespace `argocd`). There is no application code — this repo only contains Kubernetes manifests and Kustomize patches. It is meant to be applied directly (e.g. by another GitOps tool, or manually) rather than built into an artifact.

## Structure

- `kustomization.yaml` — entry point. Pulls the upstream Argo CD install manifest directly from a pinned GitHub tag (`https://raw.githubusercontent.com/argoproj/argo-cd/<tag>/manifests/install.yaml`) and layers local patches on top via `patchesStrategicMerge`.
- `overlays/configmap.yaml` — strategic-merge patch for `argocd-cm` (adds Helm repositories Argo CD should know about, e.g. HashiCorp's chart repo).

There is currently no ingress/route exposing `argocd-server` outside the cluster — access it via `kubectl port-forward` or similar. (A Traefik `IngressRoute` was previously added and then removed; see git history around `overlays/ingress.yaml` if re-adding external access, including the gRPC-header-routing gotcha needed for the Argo CD CLI.)

## Working in this repo

- **Upgrading Argo CD**: bump the version tag embedded in the `resources:` URL in `kustomization.yaml` (see git history for precedent, e.g. `3ead03b`).
- **Validating changes**: since there's no build step, validate by rendering the kustomization:
  ```
  kustomize build .
  ```
  or, if using kubectl's built-in kustomize support:
  ```
  kubectl kustomize .
  ```

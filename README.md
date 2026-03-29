<div align="center">

# 🏠 home-ops

_GitOps home Kubernetes cluster managed by Flux_

[![Kubernetes](https://img.shields.io/badge/Kubernetes-k3s-326CE5?style=flat-square&logo=kubernetes)](https://k3s.io/)
[![Flux](https://img.shields.io/badge/GitOps-Flux-5468FF?style=flat-square&logo=flux)](https://fluxcd.io/)
[![Renovate](https://img.shields.io/badge/Renovate-enabled-1A1F6C?style=flat-square&logo=renovate)](https://renovatebot.com/)

</div>

---

## Overview

This repository contains the declarative configuration for my home Kubernetes cluster running on k3s (Proxmox). All changes pushed to `main` are automatically reconciled by Flux v2.

## Hardware

| Role | Device | CPU | RAM | Storage |
|------|--------|-----|-----|---------|
| Master + Worker | Dell PowerEdge R720XD | 2x Xeon E5-2650L v2 | 32GB DDR3 ECC | 2x 2TB SAS |
| Worker | Lenovo M910Q Tiny | Intel i5-7500T | 16GB DDR4 | — |
| Worker | Chuwi Larkbox X 2023 | Intel N100 | 12GB LPDDR5 | — |

## Stack

| Component | Description |
|-----------|-------------|
| [k3s](https://k3s.io/) | Lightweight Kubernetes distribution |
| [Flux v2](https://fluxcd.io/) | GitOps operator — watches this repo and applies changes |
| [MetalLB](https://metallb.universe.tf/) | Bare-metal LoadBalancer |
| [OpenObserve](https://openobserve.ai/) | Observability platform (logs, metrics, traces) |
| [Longhorn](https://longhorn.io/) | Distributed block storage |
| [Firefly III](https://www.firefly-iii.org/) | Personal finance manager |
| [SOPS](https://github.com/getsops/sops) + [age](https://github.com/FiloSottile/age) | Secrets encryption |
| [Renovate](https://renovatebot.com/) | Automated dependency updates via k9-renovate-bot |

## Repository Structure

```
cluster/
├── base/                    # Flux entrypoints
│   ├── flux-system/         # Auto-generated Flux components (do not edit)
│   ├── repositories/helm/   # HelmRepository sources
│   ├── cluster-configs.yaml # Kustomization for cluster config (runs first)
│   ├── core.yaml            # Kustomization for core services
│   └── apps.yaml            # Kustomization for applications
│
├── core/                    # Infrastructure services
│   ├── cluster-configs/     # ConfigMap + SOPS-encrypted secrets
│   ├── namespaces/          # Pre-created namespaces
│   ├── metallb-system/      # MetalLB HelmRelease + IP pools
│   └── longhorn-system/     # Longhorn HelmRelease
│
└── apps/                    # Applications (organised by namespace)
    ├── network/
    │   └── openobserve/     # OpenObserve StatefulSet
    └── home/
        └── firefly-iii/     # Firefly III + PostgreSQL (firefly-db)

.github/
├── renovate.json            # Renovate configuration
├── renovate/                # Renovate rules (groups, labels, packageRules)
└── workflows/
    └── renovate.yaml        # GitHub Actions workflow (runs hourly)
```

## How It Works

1. Changes are pushed to `main`
2. Flux polls the repository every minute and reconciles the cluster state
3. Secrets are encrypted with SOPS + age and decrypted by Flux at deploy time
4. Renovate (k9-renovate-bot) opens PRs automatically when new versions are detected

## Reconciliation Order

```
flux-system → cluster-configs → core → apps
```

`cluster-configs` runs first to ensure variables (MetalLB ranges, etc.) are available before `core` applies them.

## Secrets Management

Secrets are encrypted using [SOPS](https://github.com/getsops/sops) with an [age](https://github.com/FiloSottile/age) key. The encrypted files follow the pattern `*.sops.yaml` and are safe to commit to the repository.

The age private key is stored in the cluster as a Kubernetes Secret (`sops-age` in `flux-system`).

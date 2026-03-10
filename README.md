# Wheelofwork Shared Workflows

Ce repo contient les workflows GitHub Actions réutilisables pour l'organisation Wheelofwork.

## Workflows disponibles

### docker-build.yml

Build et push d'images Docker vers GitHub Container Registry.

**Usage :**
```yaml
jobs:
  build:
    uses: Wheelofwork/.github/.github/workflows/docker-build.yml@main
    with:
      image-name: mon-app
      dockerfile: ./Dockerfile  # optionnel
      build-args: |             # optionnel
        ARG1=value1
        ARG2=value2
```

**Inputs :**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `image-name` | ✅ | - | Nom de l'image |
| `dockerfile` | ❌ | `./Dockerfile` | Chemin du Dockerfile |
| `context` | ❌ | `.` | Contexte de build |
| `build-args` | ❌ | `""` | Arguments de build |
| `platforms` | ❌ | `linux/amd64` | Plateformes cibles |

**Outputs :**
| Output | Description |
|--------|-------------|
| `image-tag` | Tag appliqué (`preprod` ou `latest`) |
| `full-image` | Image complète avec tag |
| `image-digest` | Digest de l'image |

**Tagging :**
- Branche `main` → tag `latest`
- Branche `develop` → tag `preprod`
- Toujours ajout du SHA du commit comme tag additionnel

### deploy-gke.yml

Déploiement sur Google Kubernetes Engine via Workload Identity Federation.

**Usage :**
```yaml
jobs:
  deploy:
    needs: build
    if: github.ref == 'refs/heads/develop'
    uses: Wheelofwork/.github/.github/workflows/deploy-gke.yml@main
    with:
      deployment-name: mon-app
```

**Inputs :**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `deployment-name` | ✅ | - | Nom du deployment K8s |
| `cluster` | ❌ | `preprod-wheelofwork` | Nom du cluster |
| `location` | ❌ | `europe-west1-b` | Zone du cluster |
| `namespace` | ❌ | `default` | Namespace K8s |

## Exemple complet

```yaml
name: Build and Deploy

on:
  push:
    branches: [develop, main]

jobs:
  build:
    uses: Wheelofwork/.github/.github/workflows/docker-build.yml@main
    with:
      image-name: mon-app

  deploy:
    needs: build
    if: github.ref == 'refs/heads/develop'
    uses: Wheelofwork/.github/.github/workflows/deploy-gke.yml@main
    with:
      deployment-name: mon-app
```

## Configuration requise

### Workload Identity Federation

Les workflows utilisent Workload Identity Federation pour l'authentification GCP.
Configuration centralisée dans `deploy-gke.yml` :
- **Pool:** `github-pool`
- **Provider:** `github-provider`
- **Service Account:** `github-deployer@wheelofwork.iam.gserviceaccount.com`

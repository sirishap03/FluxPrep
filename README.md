# FluxPrep

# What is FluxCD?
--> FluxCD is a GITOPS based continuous delivery tool for k8s.
--> FluxCD automates the deployment of your code and infra into k8s by syncing from the GIT repo.
--> It ensures that what's in GIT always running in your k8s cluster.

# GITOPS means:
--> Manages k8s manifests or Helm charts on Git repo
--> Git is source of truth
--> Any changes to the cluster must go through the GIT
--> Flux watches GIT and apply changes automatically to k8s

# FluxCD Components:
--> Source controller: watches GIT for any manifests or helm charts in repo for changes
--> Kustomize controller: applies customize YAML to the cluster
--> Helm-controller: manges helm releases
--> Notification controller: Handles alerts, webhooks [eg: slack, github]
--> Image controller: automates container image version updates in GIT

# Why FluxCD?
--> Git is the single source of truth which means entire deployment state lives in GIT
--> Automatic Reconcilation: Flux automatically monitors GIT and reconsiles your k8s cluster
--> De couples CI & CD: CI handles code building, testing, push images (eg: via jenkins, github actions);;; CD is handled by Flux which pulls changes from GIT and applies them to the k8s >>>>>>>> which results in makes the system more simpler and more modular; reduces CD pipeline complexity; avoids credentials leakage as CI don't nedd k8s access.
--> It pulls configuration from GIT into the cluster
--> Developers can deploy by just committing to GIT

# How FluxCD works?
                                                                        GIT REPO (manifests/ Helm charts)
                                                                                       |
                                                                                       |
                                                                        Flux watches GIT for changes (polls or via webhooks)
                                                                                       |
                                                                                       |
                                                                           Flux applies changes to k8s
                                                                                       |
                                                                                       |
                                                                          K8S cluster is synced with GIT


# Installation guide:-
Pre-requisites of FluxCD:
A Kubernetes cluster (Minikube, GKE, EKS, AKS, etc.)
kubectl installed and pointing to your cluster
flux CLI installed
Step -1: Install Flux CLI
curl -s https://fluxcd.io/install.sh | sudo bash
flux --version
Step - 2: Connect to GitHub and Bootstrap Flux {This will install Flux into your cluster and set up the link to your Git repo.}
flux bootstrap github \
  --owner=<your-github-username> \
  --repository=<your-repo-name> \
  --branch=main \
  --path=clusters/my-cluster \
  --personal
Step - 3: Verify Flux is running
kubectl get pods -n flux-system
flux check - check health of flux
Step - 4: Configure your application deployment:
A. Add Your App Manifests to Git
Push your Kubernetes YAML files (or Kustomize overlays or Helm values) into your Git repo under a directory like apps/my-app/.
B. Create a GitRepository Source 
             **# apps/my-app-source.yaml**
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/<your-username>/<your-repo>
  ref:
    branch: main
  ignore: |
    # exclude everything except ./apps/my-app
    /*
    !/apps/my-app/

C. Create a Kustomization to Deploy It
           **# apps/my-app-kustomization.yaml**
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 5m
  path: ./apps/my-app
  prune: true
  sourceRef:
    kind: GitRepository
    name: my-app
  targetNamespace: default

Commit and push both YAML files to your repo.
Then apply them:
kubectl apply -f apps/my-app-source.yaml
kubectl apply -f apps/my-app-kustomization.yaml
Step - 5: Monitor syncing
flux get sources git
flux get kustomizations
Check logs:
kubectl logs -n flux-system deploy/kustomize-controller







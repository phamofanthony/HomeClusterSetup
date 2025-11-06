# Overview
This doc outlines installing MetalLB through ArgoCD.

### Add files to Git
In clusters/mk1/base, create a `metallb` folder. Within it, create an `app.yaml` file and a `values.yaml` file. Copy the ones found in this Git repo.

Commit and push, and ArgoCD will do the rest.
# Kubernetes Deployment

This is a sample kubernetes deployment that uses [cert-manager](https://cert-manager.io) to provision a TLS certificate for your external domain (the manifest files use connect.example.com as a placeholder).

Included in this folder are:

- Cert Manager Issuer
- Certificate for your domain
- An Ingress rule to route TLS requests to the Connect Server
- Connect Kubernetes Service
- Connect Kubernetes Deployment
  - Deploys `connect-api` and `connect-sync`
  - Uses a shared ephemeral volume for data
  - Configures liveness and readiness checks

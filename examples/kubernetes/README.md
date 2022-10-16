# Kubernetes Deployment

This is an example Kubernetes deployment for 1Password Connect that uses [cert-manager](https://cert-manager.io) to provision a TLS certificate for your external domain.

Included in this folder are:

- [`op-connect-deployment.yaml`](op-connect-deployment.yaml): Deployment resource to create the Pod for 1Password Connect
  - `connect-sync` container to sync vault(s) from 1Password.com
  - `connect-api` container to serve 1Password Connect API
  - Shared ephemeral volume for data
- [`op-connect-service.yaml`](op-connect-service.yaml): Service resource for connecting to the above Pod
- [`op-connect-issuer.yaml`](op-connect-issuer.yaml): cert-manager ClusterIssuer resource to manage TLS certificates using [Let's Encrypt](https://letsencrypt.org/)
- [`op-connect-ingress.yaml`](op-connect-ingress.yaml): Ingress resource to route TLS requests to 1Password Connect

## Prerequisites

- 1Password Connect server credentials file (see [Quick Start](/README.md#quick-start) in the root of this repository)
- Kubernetes cluster with available [ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
- [cert-manager](https://cert-manager.io) installed on your cluster (see [Installation - cert-manager Documentation](https://cert-manager.io/docs/installation/))
- A DNS provider in which you can create public records.

## Getting started

1. If you haven't already done so, save your `1password-credentials.json` file locally.

2. Create a Kubernetes Secret for the credentials file:

    ```sh
    kubectl create secret generic op-credentials --from-file=1password-credentials.json=./1password-credentials.json
    ```

    > **Note**
    >
    > If saved with a different filename or somewhere other than the working
    > directory, replace `./1password-credentials.json` with the path to your
    > credentials file, i.e.:
    >
    > ```sh
    > kubectl create secret generic op-credentials --from-file=1password-credentials.json=path/to/credentials-file.json
    > ```

3. Clone this repository and switch to this directory:

    ```sh
    git clone https://github.com/1Password/connect.git
    cd ./connect/examples/kubernetes
    ```

4. Edit [`op-connect-issuer.yaml`](op-connect-issuer.yaml). Replace the value for `spec.acme.email` with a valid email address:

    ```yaml
    ...
    spec:
      acme:
        ...
        email: user@example.com
    ...
    ```

    Save the file.

5. Edit [`op-connect-ingress.yaml`](op-connect-ingress.yaml). Replace `connect.example.com` with the fully-qualified domain name you will use for 1Password Connect (use the same FQDN for both fields):

    ```yaml
    ...
    spec: 
      rules:
        - host: connect.example.com
    ...
    tls:
      - hosts:
        - connect.example.com
    ...
    ```

    > **Warning**
    >
    > Depending on your chosen ingress controller, you may also need to add
    > additional annotations or make other changes to this manifest.

    Save the file.

## Deploy 1Password Connect

1. Apply the manifests to your cluster:

    ```sh
    kubectl apply -f .
    ```

    > **Note**
    >
    > The `.` in this command applies all `.yaml` and `.json` files in the
    > working directory. If this includes the 1Password Connect credentials
    > file, the output returned  by `kubectl` may include something similar to:
    >
    >>```output
    >> error: error validating "1password-credentials.json":...
    >> ```
    >
    > This error can safely be ignored.

2. Get the IP address of the ingress controller:

    ```sh
    kubectl get ing/connect-ingress
    ```

    The IP address for this Ingress resource will be listed under the `ADDRESS` column.

    > **Note**
    >
    > Depending on your cluster or cloud provider, it may be several minutes
    > before an IP address is assigned. If there is no IP address listed yet,
    > run the same command again in a few minutes.

3. Create a DNS record that points to the above address. cert-manager will automatically acquire a TLS certificate on your behalf (this may take a few minutes).

## Test the connection

Confirm public access by connecting using HTTPS from outside your cluster:

> **Note**
>
> You will need to set the value of the `$OP_API_TOKEN` variable to an active token for this Connect environment.

```sh
curl \
  -X GET \
  -H "Accept: application/json" \
  -H "Authorization: Bearer $OP_API_TOKEN" \
  https://connect.example.com/v1/vaults
```

This will also intialize the connection to 1Password.com and complete the first valut data sync.

## Appendix

### Updating 1Password Connect

When a new version of 1Password Connect is released, set the image version for each of the containers to rollout a new revision:

```sh
OP_CONNECT_VERSION=1.5.7; kubectl set image deploy/op-connect \
  connect-api=1password/connect-api:$OP_CONNECT_VERSION \
  connect-sync=1password/connect-sync:$OP_CONNECT_VERSION
```

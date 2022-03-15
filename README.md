# 1Password Connect

The 1Password Connect server provides a REST API that can be used to securely access data from your 1Password account. Learn more about [setting up a Secrets Automation Workflow](https://support.1password.com/secrets-automation/)

## Quick Start

### Create Server and Access Token

You can use the [1Password CLI](https://1password.com/downloads/command-line/) to set up a 1Password Connect server and issue tokens for it.

First, set up a 1Password Connect server:

```sh
op connect server create <name> --vaults <vault>[,<vault>]
```

You'll get a `1password-credentials.json` file that you'll use to deploy the Connect server.

Issue a token:

```sh
op connect token create <token_name> --server <server_name> --vaults <vault_uuid>[,(r|w|rw)]
```

### Deploy Connect Server

Deploying 1Password Connect requires 2 containers to be running in your infrastructure.

- `1password/connect-sync`: keeps information available on the server updated with 1Password.com
- `1password/connect-api`: serves the Connect REST API

Deployment Examples:

- [Helm](https://github.com/1Password/connect-helm-charts/tree/main/charts/connect#deploying-1password-connect)
- [Docker Compose](./examples/docker/compose)
- [Kubernetes Manifest](./examples/kubernetes)
- [AWS Elastic Container Service](./examples/aws-ecs-fargate)

### Server Configuration

The following environment variable configuration options are available for both contains:
- `OP_SESSION`: path to the 1password-credentials.json file. Defaults to `~/.op/1password-credentials.json`.
- `OP_HTTP_PORT`: port used by the HTTP server. Defaults to `8080`
- `OP_LOG_LEVEL`: set the logging level of the container. Defaults to `info`.

All other configuration options are only relevant for the `password/connect-api` container:
- `OP_HTTPS_PORT`: port used by the HTTP sever when TLS is configured (see below). Defaults to `8443`.
- `OP_SYNC_TIMEOUT`: define how long to wait for initial sync to complete. Defaults to `10s`.

[More information on configuration options](docs/configuration.md)

#### TLS
By default, 1Password Connect is configured for use within a trusted network. 
It is possible to enable TLS for the connection between your application and Connect. 
This can be done by either by providing your own certificate or by letting Connect request a certificate using Let's Encrypt.

Connect will listen on the port defined by `HTTPS_PORT` (default `8443`) when TLS is enabled.

**Provide own certificate**  
Connect can use a PEM-encoded private key and certificate by setting the following two environment variables for the `connect-api` container:
- `OP_TLS_KEY_FILE`: path to the private key file.
- `OP_TLS_CERT_FILE`: path to the certificate file. This should be the full certificate chain.

**Use Let's Encrypt**  
Connect can also request a certificate from the Let's Encrypt CA. 
For this, two environment variables have to be set for the `connect-api` container:
- `OP_TLS_USE_LETSENCRYPT`: should be set to any value.
- `OP_TLS_DOMAIN`: the (sub-)domain for which to request a certificate. The DNS-records for this domain must point to the Connect server.

As long as Connect is running, its HTTPS listener must be reachable on a public IP at port 443 (either by setting `OP_HTTPS_PORT=443` or by forwarding traffic at port `443` to Connect's `OP_HTTPS_PORT`)  for the server to be able to refresh its Let's Encrypt certificate.

## Related 1Password Support Links

- [For more information and full documentation](https://support.1password.com/secrets-automation/)
- [Learn how to deploy 1Password Connect with Docker](https://support.1password.com/connect-deploy-docker/)
- [Get started with the 1Password command-line tool](https://support.1password.com/command-line-getting-started/)
- [Read the command-line tool reference guide](https://support.1password.com/command-line-reference/)

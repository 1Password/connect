# 1Password Connect

The 1Password Connect server provides a REST API that can be used to securely access data from your 1Password account. Learn more about [setting up a Secrets Automation Workflow](https://support.1password.com/secrets-automation/)

## Quick Start

### Create Server and Access Token

You can use the 1Password command-line tool to set up a 1Password Connect server and issue tokens for it.
Set up a 1Password Connect server:

```sh
op create connect server <name> --vaults <vault>[,<vault>]
```

You'll get a `1password-credentials.json` file that you'll use to deploy the Connect server.
Issue a token:

```sh
op create connect token <server_name> <token_name> --vault <vault_uuid>[,(r|w|rw)] [--vault <vault_uuid>[,(r|w|rw)]]
```

### Deploy Connect Server

Deploying 1Password Connect requires 2 containers to be running in your infrastructure.

- `1password/connect-sync`: keeps information available on the server updated with 1Password.com
- `1password/connect-api`: serves the Connect REST API

Deployment Examples:

- [Helm](https://github.com/1Password/connect-helm-charts/tree/main/charts/connect#deploying-1password-connect)
- [Docker Compose](./examples/docker/compose/README.md)
- [Kubernetes Manifest](./examples/docker/compose/README.md)
- [AWS Elastic Container Service](./examples/docker/compose/README.md)

### Server Configuration

Several environment variable configuration options are available.

- `OP_SESSION`: path to the 1password-credentials.json file
- `OP_HTTP_PORT`: port used by the HTTP Server
- `OP_LOG_LEVEL`: set the logging level of the container
- `OP_SYNC_TIMEOUT`: define how long to wait for initial sync to complete

TODO: Add TLS Configuration and debug logging details

[More information on configuration options](docs/configuration.md)

## Related 1Password Support Links

- [For more information and full documentation](https://support.1password.com/secrets-automation/)
- [Learn how to deploy 1Password Connect with Docker](https://support.1password.com/connect-deploy-docker/)
- [Get started with the 1Password command-line tool](https://support.1password.com/command-line-getting-started/)
- [Read the command-line tool reference guide](https://support.1password.com/command-line-reference/)

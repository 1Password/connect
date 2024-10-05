# [Beta] Deploy 1Password Connect Server using Docker Swarm

This example describes how to deploy 1Password Connect Server as a [stack](https://docs.docker.com/engine/swarm/stack-deploy/) using [Docker Swarm](https://docs.docker.com/engine/swarm/key-concepts/#what-is-a-swarm). The stack includes two [services](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/) (one each for the Connect API and SYNC containers), a [Docker secret](https://docs.docker.com/engine/swarm/secrets/) for the `1password-credentials.json` file, and optional secrets and configuration required only for using a custom TLS certificate.

## In this folder

- [`README.md`](./README.md): the document that you are reading. 👋😃
- [`compose.template.yaml`](./compose.template.yaml): a [Compose Specification](https://docs.docker.com/compose/compose-file/) format [Compose file](https://docs.docker.com/compose/compose-file/03-compose-file/) for 1Password Connect Server
- [`compose.lets-encrypt.yaml`](./compose.lets-encrypt.yaml): an [override configuration](https://docs.docker.com/compose/multiple-compose-files/merge/) to merge the configuration necessary for Let's Encrypt
- [`compose.http.yaml`](./compose.http.yaml): optional configuration for exposing an HTTP port on the Docker host for forwarding plain-text traffic within a private network from a public TLS termination endpoint
- [`compose.custom-tls.yaml`](./compose.custom-tls.yaml): optional configuration for enabling a self-managed TLS certificate
- [`connect-api.env`](./connect-api.env): an environment file used to customize the connect-api service
- [`connect-sync.env`](./connect-sync.env): an environment file used to customize the connect-sync service

## Overview

The open source Docker Engine tooling can be used to deploy 1Password Connect Server on any supported Linux distribution. A single-node swarm is sufficient for most deployments.

## Prerequisites

- AMD64/ARM64 VM or bare metal server with a Docker-supported Linux distribution (e.g. Ubuntu, Debian, Fedora, etc.)
- Docker Engine (see [Docker Engine installation overview](https://docs.docker.com/engine/install/#server)) installed on the Linux server
- a public DNS A record pointing to the Linux server
- SSH access to the Linux server
- [Docker Desktop](https://docs.docker.com/engine/install/#desktop) or Docker Engine installed on a machine with access to the Linux server and credentials from your 1Password account

## Get started

> [!Note]
> Before proceeding, review the [Get Started](https://developer.1password.com/docs/connect/get-started/) page from our 1Password Developer page.

Create a public DNS record that points to the public IP address of the Linux server for your Connect server if you wish to have this pubically available. For example, `op-connect-server.example.com`.

### 🛠️ Prepare the Linux server

On the Linux machine that you will be using as the Docker host for your Connect server:

1. If you haven't already done so, [install Docker Engine](https://docs.docker.com/engine/install/#server) on the Linux server. Follow the Server instructions; Docker Desktop is not needed for the Linux server.
2. Follow the [post-install steps](https://docs.docker.com/engine/install/linux-postinstall/) as noted in the documentation to enable running Docker as a non-root user and ensure that Docker Engine starts when the Linux server boots.

### 👨‍💻 Prepare your desktop

All following steps should be run on the same computer where you are already using 1Password, or another machine that can access the Linux server using SSH and has access to the `1password-credentials.json` file from the integration setup:

1. If you haven't already done so, install Docker. You can use [Docker Desktop](https://docs.docker.com/engine/install/#desktop), [install Docker Engine from binaries](https://docs.docker.com/engine/install/binaries/), or install Docker using your favourite package manager.

2. Open your preferred terminal. Clone this repository and switch to this directory:

   ```sh
   git clone https://github.com/1Password/connect.git
   cd ./connect/examples/beta/docker
   ```

3. Save the `1password-credentials.json` file from [the Automated User Provisioning setup](https://start.1password.com/developer-tools/directory/) to this working directory.

4. Open `connect-api.env` in your favourite text editor. Set the value of `OP_TLS_DOMAIN` to the fully qualififed domain name of the public DNS record for your Connect server if you wish to have this publically available, created in [Get started](#get-started). For example:

   ```dotenv
   # ...
   OP_TLS_DOMAIN=op-connect-server.example.com

   # ...
   ```

   Save the file.

5. Create a Docker context to use for connecting to the Linux server:

   _Example command:_

   ```sh
   docker context create op-connect \
       --description "1Password Connect Server Docker host" \
       --docker host=ssh://user@op-connect-server.example.com
   ```

   Copy the example command to a text editor. Replace `user` with the appropriate username for your Linux server and `op-connect-server.example.com` with the host name or IP address used for SSH access to the Linux server before running the command in your terminal.

   > **Note**
   >
   > 🔑 You can store your SSH keys in 1Password and authenticate this and other SSH workflows without writing a
   > private key to disk using the [1Password SSH agent](https://developer.1password.com/docs/ssh/get-started).

6. Switch to the Docker context you just created to connect to the Docker host:

   ```sh
   docker context use op-connect
   ```

### 🪄 Create a swarm

Run this command to create the swarm:

```sh
docker swarm init # --advertise-addr 192.0.2.1
```

> **Note**
>
> Additional nodes may be optionally added to a swarm for fault tolerance. This command adds the Linux server as the
> first (and only) node in the swarm, but Docker _requires_ a unique IP address to advertise to other nodes (even if
> no other nodes will be added). See [How nodes work](https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/)
> in the Docker documentation for more details.
>
> If multiple IP addresses are detected, Docker returns an error; uncomment the `--advertise-addr` parameter (delete
> `#`), replace the example IP (`192.0.2.1`) with the appropriate IP address, and run the command again.

## Deploy 1Password Connect server

Use the Compose template to output a canonical configuration for use with Docker Swarm and create the stack from this configuration inline. Your Connect server should automatically acquire and manage a TLS certificate from Let's Encrypt on your behalf if configured this way:

```sh
docker stack config --compose-file ./compose.template.yaml \ --compose-file ./compose.lets-encrypt.yaml |
  docker stack deploy --compose-file - op-connect
```

## Test your Connect server

Run this command to view logs from the service for the Connect server container:

```sh
docker service logs op-connect_api --raw
```

Your **Connect server URL** is based on the fully qualified domain name of the DNS record created in [Get started](#get-started). For example: `https://op-connect.example.com`. Replace `mF_9.B5f-4.1JqM` with your bearer token and `https://op-connect.example.com` with your Connect server URL to test the connection and view status information.

```sh
curl --silent --show-error --request GET --header "Accept: application/json" \
  --header "Authorization: Bearer mF_9.B5f-4.1JqM" \
  https:/op-connect.example.com/health
```

<details>
<summary>Example JSON response:</summary>

```json
{
  "name": "1Password Connect API",
  "version": "1.7.3",
  "dependencies":
    [
      {
        "service": "sqlite",
        "status": "ACTIVE",
        "message": "Connected to /home/opuser/.op/data/1password.sqlite",
      },
      {
        "service": "account_data",
        "status": "AVAILABLE",
        "message": "Account data is available",
      },
      { "service": "sync", "status": "ACTIVE" },
      {
        "service": "1Password",
        "status": "UNINITIALIZED",
        "message": "Make a request with a valid bearer token to initialize",
      },
    ],
}
```

</details>
<br />

## Update 1Password Connect server

Swarm mode in Docker Engine uses a declarative service model. Services will automatically restart tasks when updating their configuration.

Use the Docker context from [Prepare your desktop](#-prepare-your-desktop) to connect to your Docker host and manage your stack.

Update the `op-connect_api` and `op-connect_sync` services with the new image tags from the `1password/connect-api` and `1password/connect-sync`repository on Docker Hub to update your Connect server to a new version:

- https://hub.docker.com/r/1password/connect-api/tags
- https://hub.docker.com/r/1password/connect-sync/tags

```sh
docker service update op-connect_api --image 1password/connect-api:1.7.3 && \
   docker service update op-connect_sync --image 1password/connect-sync:1.7.3

```

## Rotate credentials

Docker secrets are immutable and cannot be removed while in use by a Swarm service. To use new secret values in your stack, you must remove the existing secret from the service configuration, replace the Docker secret (or add a new one), and update the service configuration to mount the new secret value.

For example, if you create new credentials for the Connect server:

1. Scale down the `op-connect_api` and `op-connect_sync services to shut down the running Connect server.

   ```sh
   docker service scale op-connect_api=0 op-connect_sync=0
   ```

2. Update the service definition to unnmount the the `1password-credentials.json` Docker secret:

   ```sh
   docker service update --secret-rm credentials op-connect_api && \
      docker service update --secret-rm credentials op-connect_sync
   ```

3. Remove the secret from the swarm:

   ```sh
   docker secret rm credentials
   ```

4. Copy the new `1password-credentials.json` file from your 1Password account to your working directory. Create a new `credentials` Docker secret using the new file:

   ```sh
   docker secret create credentials ./1password-credentials.json
   ```

5. Update the service to mount the new `1password-credentials.json` secret:

   ```sh
   docker service update --secret-add source=credentials,target=/home/opuser/.op/1password-credentials.json,uid="999",gid="999",mode=0440 op-connect_api && \
      docker service update --secret-add source=credentials,target=/home/opuser/.op/1password-credentials.json,uid="999",gid="999",mode=0440 op-connect_sync

   ```

6. Scale the `op-connect_api` and `op-connect_sync` services back up to reboot your Connect server:

   ```sh
   docker service scale op-connect_api=1 op-connect_sync=1
   ```

[Test your Connect server](#-test-your-Connect server) using the new bearer token associated with the new `1password-credentials.json` file.

A similar process can be used to update the values for any other Docker secrets used in your configuration.

## Appendix: Customize your Connect server

Many Connect server configuration changes can be made by adding or removing environment variables. These can be customized by making changes to [`connect-api.env`](./connect-api.env) and [`connect-sync.env`](./connect-sync.env) (that can be commited to your source control) before deploying (or redeploying) your Connect server using the `docker stack deploy` command. For some use cases, it may be desirable to update the configuration "on the fly" from your terminal.

For example, to reboot your Connect server with debug logging enabled:

```sh
docker service update op-connect_api \
    --env-add OP_DEBUG=1
```

To turn off debug logging and inject some colour into the logs in your console:

```sh
docker service update op-connect_api \
    --env-rm OP_DEBUG \
    --env-add OP_PRETTY_LOGS=1
```

_Pretty logs pair nicely with the `--raw` parameter of the `docker service logs` command). 🤩_

### 🔒 Advanced TLS options

Identity providers strictly require an HTTPS endpoint with a vlid TLS certificate to use for the Connect server URL. Your 1Password Connect server includes an optional CertificateManager component that (by default) acquires and manages a TLS certificate using Let's Encrypt, and terminates TLS traffic at the Connect server container using this certificate. This requires port 443 of the Docker host to be publicly accessible to ensure Let's Encrypt can initiate an inbound connection to your Connect server.

Other supported options include:

#### External load balancer or reverse proxy

To terminate TLS traffic at another public endpoint and redirect private traffic to a Docker host in your private network, a Connect server can be configured to disable the CertificateManager component and serve plain-text HTTP traffic on port 80 of the Docker host. CertificateManager will be enabled if a value is set for the `OP_TLS_DOMAIN` variable, so any value set in `connect-api.env` must be removed (or this line must be commented out). The included `compose.http.yaml` file can be used to set up the port mapping when deploying (or redeploying) a Connect server:

```sh
docker stack config \
    --compose-file ./compose.template.yaml \
    --compose-file ./compose.http.yaml |
        docker stack deploy --compose-file - op-connect
```

#### Self-managed TLS certificate

You may supply your own TLS certificate with CertificateManager instead of invoking Let's Encrypt. The value set for `OP_TLS_DOMAIN` must match the common name of the certificate.

Save the public and private certificate key files as `certificate.pem` and `key.pem` (respectively) to the working directory and use the included `compose.custom-tls.yaml` file when deploying your Connect server to create Docker secrets and configure a Connect server use this certificate when terminating TLS traffic:

```sh
docker stack config \
    --compose-file ./compose.template.yaml \
    --compose-file ./compose.custom-tls.yaml |
        docker stack deploy --compose-file - op-connect
```
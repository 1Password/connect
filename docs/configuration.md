# Connect Server Configuration

The Connect server consists of 2 containers, `1password/connect-api` and `1password/connect-sync` running in the same network. The two containers require a shared volume to store an encrpyted copy of your data.

## Data Volume

- `/home/opuser/.op/data`: The default location of the shared volume

**Note**: If you use are setting the `XDG_DATA_HOME` environment variable to a path other than `/home/opuser` you will need to mount your data volume at that path as well.

## Environment variables

The following environment variable configuration options are available for both contains:

- `OP_SESSION`: path to the 1password-credentials.json file
- `OP_HTTP_PORT`: port used by the HTTP server
- `OP_LOG_LEVEL`: set the logging level of the container
- `XDG_DATA_HOME`: set the path where the `.op/data` directory should be created
- `OP_BUS_PORT`: the port used for listening to incoming bus connections from other containers (by default, this is a random free port)
- `OP_BUS_PEERS`: a comma-separated listed of `[hostname]:[bus port]` pairs of other containers to connect to (see _Manual bus configuration_ for more details)

All other configuration options are only relevant for the `1password/connect-api` container:

- `OP_HTTPS_PORT`: port used by the HTTP sever when TLS is configured (see below)
- `OP_SYNC_TIMEOUT`: define how long to wait for initial sync to complete

When using TLS with own certificate:

- `OP_TLS_KEY_FILE`: path to the private key file.
- `OP_TLS_CERT_FILE`: path to the certificate file. This should be the full certificate chain.

When using TLS with Let's Encrypt:

- `OP_TLS_USE_LETSENCRYPT`: should be set to any value.
- `OP_TLS_DOMAIN`: the (sub-)domain for which to request a certificate. The DNS-records for this domain must point to the Connect server.

## Manual bus configuration (v1.5.0 and above)
By default, the 2 containers automatically discover and connect to the shared bus. This discovery mechanism requires the `NET_BROADCAST` capability, which cannot always be granted to containers.

It is also possible to manually configure the shared bus:
1. Assign a static port for the bus by setting the `OP_BUS_PORT` environment variable to a free port for all containers.
2. Set the `OP_BUS_PEERS` environment variable for the api container to `[hostname]:[bus port]`, where hostname is the name of the sync container and bus port the value of `OP_BUS_PORT` for the sync container.
3. Set the `OP_BUS_PEERS` environment variable for the sync container to `[hostname]:[bus port]`, where hostname is the name of the api container and bus port the value of `OP_BUS_PORT` for the api container.

For example, if the containers are called `op-connect-api` and `op-connect-sync`. The configuration of `op-connect-api` would be:
```
OP_BUS_PORT=11223
OP_BUS_PEERS=op-connect-sync:11223
```
And for `op-connect-sync`:
```
OP_BUS_PORT=11223
OP_BUS_PEERS=op-connect-api:11223
```

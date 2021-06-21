# Connect Server Configuration

The Connect server consists of 2 containers, `1password/connect-api` and `1password/connect-sync` running in the same network. The two containers require a shared volume to store an encrpyted copy of your data.

## Data Volume

- `/home/opuser/.op/data`: The location of the shared volume

## Environment variables

The following environment variable configuration options are available for both contains:
- `OP_SESSION`: path to the 1password-credentials.json file
- `OP_HTTP_PORT`: port used by the HTTP server
- `OP_LOG_LEVEL`: set the logging level of the container

All other configuration options are only relevant for the `password/connect-api` container:
- `OP_HTTPS_PORT`: port used by the HTTP sever when TLS is configured (see below)
- `OP_SYNC_TIMEOUT`: define how long to wait for initial sync to complete

When using TLS with own certificate:
- `OP_TLS_KEY_FILE`: path to the private key file.
- `OP_TLS_CERT_FILE`: path to the certificate file. This should be the full certificate chain.

When using TLS with Let's Encrypt:
- `OP_TLS_USE_LETSENCRYPT`: should be set to any value.
- `OP_TLS_DOMAIN`: the (sub-)domain for which to request a certificate. The DNS-records for this domain must point to the Connect server.

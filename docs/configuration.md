# Connect Server Configuration

The Connect server consists of 2 containers, `1password/connect-api` and `1password/connect-sync` running in the same network. The two containers require a shared volume to store an encrpyted copy of your data.

## Data Volume

- `/home/opuser/.op/data`: The location of the shared volume

## Environment variables

- `OP_SESSION`: path to the 1password-credentials.json file
- `OP_HTTP_PORT`: port the container is available on
- `OP_LOG_LEVEL`: set the logging level of the container
- `OP_SYNC_TIMEOUT`: define how long to wait for initial sync to complete

TODO: Add TLS Configuration and debug logging details

# Docker Compose

The `docker-compose.yaml` file in this directory is a simple example of how to run 1Password Connect using Docker Compose.

## Getting Started

Before you begin, make sure you have your Connect server's `1password-credentials.json` file on your machine. If the credentials file is not located in the same directory as the `docker-compose.yaml` file, update the `volumes` section for both the Sync and API containers:

```yaml
volumes:
  - "<filepath_here>/1password-credentials.json:/home/opuser/.op/1password-credentials.json"
```

### Start the Docker containers:
```bash
docker-compose up [-d]
```

The Connect API will be accessible through port `8080` on the host machine:
```curl
curl \
-H "Accept: application/json" \
-H "Authorization: Bearer $OP_API_TOKEN" \
http://localhost:8080/v1/vaults
```
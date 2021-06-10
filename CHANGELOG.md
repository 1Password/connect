[//]: # "START/v1.2.0"

# v1.2.0

This release contains several improvements to securing you API requests and better error handling when dealing with Items.

## Features

- [NEW] Add TLS for the Connect API. Provide your own certificates or use LetsEncrypt

- [IMPROVED] A more user-friendly error is returned when the user attempts to create an item with the "CUSTOM" category. {1382}

## Fixed

- [FIXED] Resolved issue where API could incorrectly report a successful item deletion. {1368}
- [FIXED] Fixed the handling of TOTP fields for items in Connect {1372}

[//]: # "START/v1.1.1"

# v1.1.1

This release contains improvements to account synchronization and token validation.

[//]: # "START/v1.1.0"

# v1.1.0

This release contains several improvements to configuring, running, and troubleshooting a Secrets Automation deployment in your environment.

The health endpoint returns additional information about the status of the local item database and the "DELETE" endpoint now permanently deletes items instead of moving them to the Trash.

## Features

- [NEW] Updated the Connect API to delete items instead of trashing them. {11613}
- [NEW] Connect can now also be unlocked by sending a bearer token to the /health endpoint. {1305}
- [NEW] The /health endpoint reports the status of the data in the local database under account_data. {1305}
- [NEW] Added support for new API Credential item category. {1344}

- [IMPROVED] The `OP_LOG_LEVEL` evironment variable can now be used to customize the logging level of the Connect server and the sync component. {1205}
- [IMPROVED] The API request timeout for waiting for a healthy sync is now per-request and can be changed with the `OP_SYNC_TIMEOUT` environment variable. {1305}
- [IMPROVED] Item usage is now reported to 1password.com in batches at most once a second to avoid being rate limited. {1331}
- [IMPROVED] Syncer applies an exponential backoff when retrying if it cannot find a valid credential. {1341}

## Fixes

- [FIXED] Corrected several minor mistakes in the OpenAPI spec. {1326}
- [FIXED] Log error when a connection problem causes the failure of a Connect request. {1302}
- [FIXED] Updating the password for a Login or Password Item correctly updates the Password History. {1350}

[//]: # "START/v1.0.0"

# v1.0.0

The 1Password Connect server provides applications and services with information from 1Password as part of 1Password Secrets Automation.

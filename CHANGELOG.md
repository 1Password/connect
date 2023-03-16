[//]: # "START/v1.6.0"

# v1.6.0
This release moves Connect to a distroless base image.
- [SECURITY] Connect Docker images  are now based on gcr.io/distroless/base to reduce the number of false positives when scanned with container security tools. {3296}

[//]: # "START/v1.5.8"

# v1.5.8
This release contains two security improvements.
- [SECURITY] Addressed an issue that allows an attacker with network access to the Connect server to allocate high amounts of memory on the Connect server if it is configured to serve connections over TLS. We inherited this issue from the Go standard library, where the issue has been assigned vulnerability identifier CVE-2022-41717. {3274}
- [SECURITY] Update Debian packages for the Docker release images. {3195}

[//]: # "START/v1.5.7"

# v1.5.7
Security updates, and a fix to allow Connect to (re)start if the 1Password server is unavailable.
- [FIXED] The API container will now continue to serve requests if it is restarted when 1Password server in unreachable. {2560}

- [SECURITY] Update connect images to use latest base image. {2738}
- [SECURITY] Update Debian packages for the Docker release images. {2694}

[//]: # "START/v1.5.6"

# v1.5.6
This version addresses bugs related to file objects

- [FIXED] The file objects now have the correct ID and name. {2625, 2616}
- [FIXED] Categories route will no longer return status 500 when no error is present. {2170}

[//]: # "START/v1.5.5"

# v1.5.5
This version addresses a few bugs and a security improvement.

- [FIXED] Connect now properly returns the `content_path` attribute of the File object. {2598}

- [SECURITY] Updated Debian packages for the Docker release images. {2516}

[//]: # "START/v1.5.4"

# v1.5.4
This release updates the API spec of Connect to mention some supported fields and addresses two bugs.

- [IMPROVED] The API spec now mentions the "label" field for autofill URL's. {2446}
- [IMPROVED] The API spec now mentions support for the item categories "MEDICAL_RECORD" and "SSH_KEY". {2468}

- [FIXED] Connect accepts JSON inputs with camelCase keys as used in the SDK's again. {2317}
- [FIXED] Connect no longer return an error mentioning the files directory in some cases when a file is uploaded to a vault that Connect syncs. {2498,2539}

- [SECURITY] The Docker images are now using an updated version of the Debian base images. {2516}

[//]: # "START/v1.5.3"

# v1.5.3
This release contains improvements in validation during the SRP process.

- [SECURITY] Improved validation of server parameters in SRP process. Credits to Cure53. {2442}

[//]: # "START/v1.5.2"

# v1.5.2
This release addressed a few minor bugs and a memory leak.

- [IMPROVED] Connect now validates `monthYear` and `date` field types. {2075}

- [FIXED] Addressed a problem that could let the memory consumption of the sync container slowly increase for every incoming request under certain circumstances. {2017}
- [FIXED] Having a single quote as a filter value no longer throws a stack trace and now returns a 400. {1442}

[//]: # "START/v1.5.1"

# v1.5.1
This release addresses a few problems related to Connect's synchronization mechanism.

- [IMPROVED] Connect stays better in sync with 1Password when it experiences connection problems. {2150}

- [FIXED] Fixed synchronization of templates that could prevent Connect from starting after changing its credentials. {2151}
- [FIXED] Templates should now correctly synchronize to Connect after emptying Connect's database. {2151}
- [FIXED] Connect should no longer return a "failed to initialize database" error when upgrading from Connect v1.3.x. {2005}

[//]: # "START/v1.5.0"

# v1.5.0
This release introduces Docker images for the `arm64` and `arm/v7` platforms. 

Furthermore, the bus between the api and sync container can now be manually configured for environments that cannot run containers with the `NET_BROADCAST` capability.

- [NEW] Bus auto-discovery can now be disabled by explicitly defining peers with the `$OP_BUS_PEERS` environment variable. {1778}
- [NEW] Connect API and Connect Sync Docker images for arm64/v8 and arm/v7 architectures are now available. {1771}

- [FIXED] The label field is now always correctly set (instead of being empty) for fields that have a USERNAME or PASSWORD purpose. {1657}



[//]: # "START/v1.4.0"

# v1.4.0
This release introduces enhancements for OTP fields, generated passwords and item usage histories.

This update includes non-breaking changes to the  [OpenAPI spec](docs/openapi/spec.yaml).

- [NEW] Specific characters can now be excluded from generated passwords by setting `excludeCharacters` in the recipe. {1631}
- [IMPROVED] Added the generated TOTP code to the body of OTP fields in Items. {1557}
- [IMPROVED] Connect now records more detailed item usage histories. {14198}


[//]: # "START/v1.3.0"

# v1.3.0

This release introduces support for accessing documents and files attached to items.

## Features

- [NEW] Add support for getting files through the API. Details can be found in the [OpenAPI spec](docs/openapi/spec.yaml).

- [IMPROVED] Item modification requests are now rejected by the Connect API if the authenticated account is in the frozen billing state. {1175}
- [IMPROVED] Items containing one or more file fields can now be updated through Connect. {1386}
- [IMPROVED] API spec now defines valid vault and item IDs. {1399}
- [IMPROVED] Connect API now reports its version number in the `1Password-Connect-Version` header. {1443}


## Fixed

- [FIXED] Resolved an issue that prevented servers from authenticating with accounts after an admin updated the sign-in url. {1446}


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

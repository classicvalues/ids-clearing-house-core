# IDS Clearing House Core
The IDS Clearing House Core consists of two microservices that support the [Clearing House Service](https://github.com/Fraunhofer-AISEC/ids-clearing-house-service), which is a prototype implementation of the Clearing House component of the [Industrial Data Space](https://github.com/International-Data-Spaces-Association/IDS-G). The Clearing House provides an API to store and retrieve data. Data in the Clearing House is stored encrypted and practically immutable.

## DAPS
The Clearing House Services (i.e. both Document API and Keyring API) need to be able to validate the certificate used by the DAPS. If the DAPS uses a self-signed certificate the certificate needs to be added in two places:
1. `/server/certs`: The Clearing House App will load certificates in this folder in the container and use them for validation. The certificate needs to be in DER format.
2. `/usr/local/share/ca-certificates`: The Clearing House App relies on openssl for parts of the validation and openssl will not trust a self-signed certificate unless it was added in this folder and `update-ca-certificates` was called in the docker container. Once this is done the container might need to be restarted.

If you are using the pre-build docker containers and use `daps.aisec.fraunhofer.de` as the DAPS, only Step 1 is required.

## Document API
The Document API is responsible for storing the data and performs basic encryption and decryption for which it depends on the Keyring API.

### Configuration
The Document API is configured using the configuration file `Rocket.toml`, which must specify a set of configuration options, such as the correct URLs of the database and other service apis:
- `daps_api_url`: Specifies the URL of the DAPS Service. Required to validate DAPS token
- `keyring_api_url`: Specifies the URL of the Keyring API
- `database_url`: Specifies the URL of the database to store the encrypted documents. Currently only mongodb is supported so URL is supposed to be `mongodb://<host>:<port>`
- `clear_db`: `true` or `false` indicates if the database should be cleared when starting the Service API or not. If `true` a restart will wipe the database! Starting the Service API on a clean database will initialize the database.

When starting the Clearing House Service API it also needs the following environment variables set:
- `API_LOG_LEVEL`: Allowed log levels are: `Off`, `Error`, `Warn`, `Info`, `Debug`, `Trace`

## Keyring API
The Keyring API is responsible for creating keys and the actual encryption and decryption of stored data.

### Configuration
The Keyring API is configured using the configuration file `Rocket.toml`, which must specify a set of configuration options, such as the correct URLs of the database and other service apis:
- `daps_api_url`: Specifies the URL of the DAPS Service. Required to validate DAPS token
- `database_url`: Specifies the URL of the database to store document types and the master key. Currently only mongodb is supported so URL is supposed to be `mongodb://<host>:<port>`
- `clear_db`: `true` or `false` indicates if the database should be cleared when starting the Service API or not. If `true` a restart will wipe the database! Starting the Service API on a clean database will initialize the database.

When starting the Clearing House Service API it also needs the following environment variables set:
- `API_LOG_LEVEL`: Allowed log levels are: `Off`, `Error`, `Warn`, `Info`, `Debug`, `Trace`

The Keyring API requires that its database contains the acceptable document types. Currently only the IDS_MESSAGE type is supported and needs to be present in the database for the Keyring API to function properly. The database will be populated with an initial document type that needs to be located in `init_db/default_doc_type.json`.
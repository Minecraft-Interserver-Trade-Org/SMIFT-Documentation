SMIFT Certificates are cryptographic "documents" issued by a SMIFT Network to identify and authenticate a node.

# Version History
- [SMIFT Certificate 1.0](#version-1.0)

# Version 1.0
## Technology
SMIFT Certificates use RSA asymmetric cryptography to authenticate nodes. SMIFT Certificates are **NEVER** exchanged directly through SMIFT.

## Encoding
SMIFT Certificates use base 64 representation of a UTF-8 JSON object.

## Data types
| Type | Description |
| - | - |
| `TEXT` | Plain UTF-8 text |
| `DATE` | Date using the format `YEAR-MONTH-DAY` and timezone `UTC+00:00` |
| `KEY` | UTF-8 Representation of an RSA key |
| `VERSION` | SMIFT Certificate version represented as UTF-8 text using the format `MAJOR.MINOR` |

## Contents
SMIFT Certificates hold the following information
| Field | Type | Description |
| - | - | - |
| `Certificate-Version`| `VERSION` | The SMIFT Certificate version used for this certificate |
| `Server-Name` | `TEXT` | Name of the server to which the node belongs |
| `Issuance-Date` | `DATE` | Date of issuance of the certificate |
| `Expiration-Date` | `DATE` | Expiration date of the certificate |
| `Issuer-Name` | `TEXT` | The issuing network's name |
| `Issuer-Key`| `KEY` | The RSA public key associated with the issuing network |
| `Public-Key` | `KEY` | The RSA public key associated with this node |
| `Private-Key` | `KEY` | The RSA private key associated with this node |

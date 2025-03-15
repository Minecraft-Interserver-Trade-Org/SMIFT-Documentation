The SMIFT Protocol, or _SMIFTP_ for short, is a set of rules used by clients and servers to communicate on a SMIFT Network. The SMIFT Protocol defines the type and structure of messages, alongside timings, accepted values and other technical details.

## History
Development on SMIFTP 0.1 started on May 12th, 2024. This version defines the basic format for SMIFT messages and a few fundamental primitives.

## Technology
Messages in this version of the SMIFT Protocol are sent via TCP through an unsecured channel.

## Encoding
Messages in this version of the SMIFT Protocol use [UTF-8](https://en.wikipedia.org/wiki/UTF-8) encoding and are zero-terminated. Lines are terminated by the CRLF sequence.

## Glossary
Certain terms used in this page may be ambiguous. When reading, keep in mind this table:
| Term | Meaning |
| - | - |
| Server | Minecraft/Game Server|
| SMIFT Routing Server | The server software acting as SMIFT middleman |
| Transaction | Any SMIFT operation that needs to be forwarded to a third node, such as a transfer or message |
| Node | Programme used by Minecraft/Game server owners to interact with SMIFT |
| Client | Node that is requesting a service, such as a transfer |

## Message sequence
SMIFTP is a fully asynchronous protocol. Exchanges generally start with a request from a client and end with a response from another client. The SMIFT Routing Server acts as a middleman between nodes, forwarding messages from one to another. Due to the asynchronous nature of SMIFTP, the Routing Server and clients use the initial request hash, unique transaction ID, and state machines to keep track of requests.

## Message format
Messages in SMIFTP 0.1 are comprised of a header and a body, separated by an empty line. The message body contains attributes in a STOMP-like syntax.
Headers in this version of SMIFTP contain a version phrase, message type and additional attributes.

```
SMIFT/0.1 <MESSAGE TYPE> <ARGUMENTS>

<KEY>: <VALUE>
...
<KEY>: <VALUE>
```

Following is an example of a message valid in SMIFTP 0.1
```
SMIFT/0.1 RESPONSE 500 Hello world

Property-X: hello
Property-Y: world
Timestamp: 2024-05-12 00:00:00
```

## Data types
Body attributes can have one of several data types:
| Type | Description |
| - | - |
| `INT`| Positive integer |
| `TEXT` | Text string that contains a maximum of 200 characters. All non-alphanumeric characters are escaped using `\` |
| `SSID` | SMIFT Server Identifier |
| `SCC` | ISO 4217-like SMIFT Currency Code |
| `STID` | SMIFT Transaction ID |
| `SHA256` | UTF-8 representation of SHA256 hash |
| `TIMESTAMP` | Date and time using the syntax `YEAR-MONTH-DAY HOURS:MINUTES:SECONDS` |
| `LIST` | List of one of the above types. Uses comma separator |

## Message types
Messages can be one of three types
- `REQUEST`
- `RESPONSE`
- `FORWARD`

### Request messages
Requests are initiated by a client looking to communicate with another client and use the following header:
```
SMIFT/0.1 REQUEST <COMMAND>
```

Request messages **ALWAYS** have a `Timestamp` field in the body. This timestamp is used to store the time and date on which the request was originally generated and is of type `TIMESTAMP`. This is useful to ensure that message hashes are always different.

The full list of commands can be found in the [dedicated section][#Rquest-commands].

### Response messages
Response messages are either sent from the Routing Server to a requesting client, or from a handling client to the Routing Server. These messages use the following header:
```
SMIFT/0.1 RESPONSE <CODE> <PHRASE>
```

Response messages **ALWAYS** have a `Timestamp` field in the body. This timestamp is used to store the time and date on which the response was originally generated and is of type `TIMESTAMP`. This is useful for deferred responses.

Response codes and phrases can be found in the [dedicated section][#Response-messages].

### Forward messages
Forward messages are used by the SMIFT Routing Server to forward an incoming request to a handler node. Forward messages differ from request messages in commands and body contents, but use a similar header:
```
SMIFT/0.1 FORWARD <COMMAND>
```

Forward messages **ALWAYS** have a `Timestamp` field in the body. This timestamp is used to store the time and date on which the request was originally generated and is of type `TIMESTAMP`. This is useful for deferred requests.

The full list of commands can be found in the [dedicated section](#Forward-commands).


## Request commands
Request messages can be used to issue the following commands:
| Name | Code | Description |
| -    | -    | -           |
| Authenticate | `AUTHENTICATE` | Authenticates a newly connected node |
| Transfer | `TRANSFER` | Issues an order to transfer funds |
| Cancel transfer | `CANCEL TRANSFER` | Issues an order to cancel an unsettled transfer |


### Authenticate
This is the first message a client sends to the SMIFT Routing Server after establishing a connection.
| Attribute | Type | Description |
| - | - | - |
| `Server-Identifier` | `SSID` | Specifies the Game Server's SMIFT Server Identifier |
| `Password` | `TEXT` | Specifies the user's password |

The SMIFT Routing Server can reply with one of the following responses: `200`, `300`, `301`, `302`.

### Transfer
This command can only be issued by authenticated clients.
| Attribute | Type | Description |
| - | - | - |
| `Amount` | `INT` | The amount to be transferred |
| `Currency` | `SCC` | The currency used for the transfer |
| `Destination-Server` | `SSID` | The payee's server |
| `Destination-Account` | `TEXT` | The payee's account identifier |
| `Note` | `TEXT` | A note to help identify this transfer |

The server can reply with one **or more** of the following responses: `110`, `111`, `112`, `210`, `301`, `302`, `303`.

**NOTE:** `112`, `210` responses may be deferred due to settlement and clearing delays.

### Cancel transfer
This command can only be issued by authenticated clients.
| Attribute | Type | Description |
| - | - | - |
| `Transaction-ID` | `STID` | The transfer to be cancelled |

The server can reply with one of the following responses: `113`, `211`, `301`, `302`, `303`.

**NOTE:** `211` responses may be deferred due to settlement and clearing delays.

## Response messages
Response messages can be classified as:
- Progress
- Complete
- Error

The second digit in the response code identifies the response's subtype. Response subtypes are common across response classes
| Code | Subtype |
| - | - |
| X0X | Request/Client |
| X1X | Transfer |

### Progress
`Progress` responses use codes `100-199` and are used to communicate updates about a transaction.

#### 110 Transfer routed
| Attribute | Type | Description |
| - | - | - |
| `Request-Hash` | `SHA256` | The SHA256 hash of the initial request body |
| `Transaction-ID` | `STID` | The unique identifier used for this transfer |

#### 111 Transfer authorised
This response is generated after a transfer request has been accepted (but not settled) by the handler node.
| Attribute | Type | Description |
| - | - | - |
| `Transaction-ID` | `STID` | The unique identifier used for this transfer |

#### 112 Transfer settled
This response is generated after a transfer request has been settled by the handling node.
| Attribute | Type | Description |
| - | - | - |
| `Transaction-ID` | `STID` | The unique identifier used for this transfer |


#### 113 Transfer cancelled by SMIFT Routing Server
This response is sent after the SMIFT Routing Server has authorised the cancellation of a transfer, but **BEFORE** the authorisation by the handling node
| Attribute | Type | Description |
| - | - | - |
| `Request-Hash` | `SHA256` | The SHA256 hash of the initial request body |
| `Target-Transaction-ID` | `SHA256` | The unique identifier of the cancelled transfer |
| `Transaction-ID` | `STID` | The unique identifier used for the cancellation |


### Complete
`Complete` responses use codes `200-299` and are used to indicate that a transaction has been completed. Once a `Complete` response is sent, the transaction, if present, is considered closed.

#### 200 Authenticated
This message is sent by the SMIFT Routing Server as a positive response to an `AUTHENTICATE` request.

#### 210 Transfer credited
This response is sent after a transfer has been settled by the handling node and credited to the final user.
| Attribute | Type | Description |
| - | - | - |
| `Transaction-ID` | `STID` | The unique identifier used for this transfer |

#### 211 Transfer cancelled
This response is sent after a transfer has been cancelled by the handling node.
| Attribute | Type | Description |
| - | - | - |
| `Target-Transaction-ID` | `SHA256` | The unique identifier of the cancelled transfer |
| `Transaction-ID` | `STID` | The unique identifier used for the cancellation |

### Error
`Error` responses use codes `300-399` and are used to communicate a failure. Once an `Error` response is sent, the transaction, if present, is aborted.

#### 300 Unsupported SMIFTP version
This response is sent by the SMIFT Routing Server if a client is using an unsupported version.
| Attribute | Type | Description |
| - | - | - |
| `Oldest-Supported` | `TEXT` | The oldest supported SMIFTP version by the SMIFT Routing Server  |
| `Latest-Supported` | `TEXT` | The newest supported SMIFTP version by the SMIFT Routing Server |

#### 301 Access denied
This response is sent if an unauthenticated client is trying to execute a request, or if a node refuses a request bases solelly on the client's identity (for example, in the case of econoimc sanctions).

#### 302 Malformed request
This response is sent if the initial request contained syntax errors.

#### 303 Invalid attributes
This response is sent if one or more attributes of the initial request are invalid (i.e. uknown destination server, currency, account number, etc).

## Forward commands
Forward messages can be used to issue the following commands:
| Name | Code | Description |
| -    | -    | -           |
| Transfer | `TRANSFER` | Issues an order to transfer funds |
| Cancel transfer | `CANCEL TRANSFER` | Issues an order to cancel an unsettled transfer |
| Credit | `CREDIT` | Issues an order to credit funds to a user account |

### Transfer
This command is issued by the SMIFT Routing Server and sent to a handling node for settlement.
| Attribute | Type | Description |
| - | - | - |
| `Transaction-ID` | `STID` | The ID associated by the SMIFT Routing Server with this transaction |
| `Amount` | `INT` | The amount to be transferred |
| `Currency` | `SCC` | The currency used for the transfer |
| `Destination-Server` | `SSID` | The payee's server |
| `Sender-Server` | `SSID` | The payer's server |
| `Destination-Account` | `TEXT` | The payee's account identifier |

### Cancel transfer
This command is issued by the SMIFT Routing Server and sent to a handling node to complete the cancellation.
| Attribute | Type | Description |
| - | - | - |
| `Transaction-ID` | `STID` | The transfer to be cancelled |

### Credit
This command is issued by the SMIFT Routing Server and sent to the final handling node to credit the final user's account.
| Attribute | Type | Description |
| - | - | - |
| `Transaction-ID` | `STID` | The ID associated by the SMIFT Routing Server with this transaction |
| `Amount` | `INT` | The amount to be transferred |
| `Currency` | `SCC` | The currency used for the transfer |
| `Destination-Account` | `TEXT` | The payee's account identifier |
| `Note` | `TEXT` | A note to help identify this transfer |

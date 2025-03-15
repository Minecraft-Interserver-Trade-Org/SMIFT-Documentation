The SMIFT Protocol, or _SMIFTP_ for short, is a set of rules used by clients and servers to communicate on a SMIFT Network. The SMIFT Protocol defines the type and structure of messages, alongside timings, accepted values and other technical details.

## History
Development on SMIFTP 0.2 started on August 30th, 2024. This version builds on SMIFTP 0.1 and introduces special features to facilitate the operation of wholsale foreign exchange markets over SMIFT.

**NOTE:** This version has not been finalised yet.

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
Messages in SMIFTP 0.2 are comprised of three sections separed by empty lines:
- The `Header` section is a single line stating a version phrase, the message type and additional attributes
- The `Body` section contains important message attributes in a STOMP-like syntax
- The optional `Payload` section contains additional information in JSON format

```
SMIFT/0.2 <MESSAGE TYPE> <ARGUMENTS>

<KEY>: <VALUE>
...
<KEY>: <VALUE>

[<PAYLOAD>]
```

The contents exchanged in the `Body` and `Payload` sections are detailed in the "Attributes" and "Paylaod" sections for each [request](#request-commands), [response](#response-messages), and [forward](#forward-commands). Messages that do not support one or both of these sections lack the dedicated paragraphs in their documentation. **NOTE:** Some generic attributes are defined in the [message types section](#message-types).

Following is an example of a valid message in SMIFTP 0.2
```
SMIFT/0.2 RESPONSE 500 Hello world

Property-X: hello
Property-Y: world
Timestamp: 2024-05-12 00:00:00

{ "message": "Hello, world!" }
```

## Data types
Body attributes can have one of several data types:
| Type | Description |
| - | - |
| `INT`| Positive integer |
| `DECIMAL` | Decimal value |
| `TEXT` | Text string that contains a maximum of 200 characters. All non-alphanumeric characters are escaped using `\` |
| `SHORT` | Text string that contains a maximum of 24 characters. All non-alphanumeric characters are escaped using `\` |
| `SSID` | [SMIFT Server Identifier](https://github.com/Alessandro-Salerno/SMIFT/wiki/SMIFT-Codes-&-Symbols-Specification-1.0#smift-server-identifier-ssid) |
| `SCC` | [SMIFT Currency Code](https://github.com/Alessandro-Salerno/SMIFT/wiki/SMIFT-Codes-&-Symbols-Specification-1.0#smift-currency-code-scc) |
| `STID` | [SMIFT Transaction Identifier](https://github.com/Alessandro-Salerno/SMIFT/wiki/SMIFT-Codes-&-Symbols-Specification-1.0#smift-transaction-identifier-stid) |
| `SAID` | [SMIFT Account Identifier](https://github.com/Alessandro-Salerno/SMIFT/wiki/SMIFT-Codes-&-Symbols-Specification-1.0#smift-account-identifier-said) |
| `SHA256` | UTF-8 representation of SHA256 hash |
| `TIMESTAMP` | Date and time using the syntax `YEAR-MONTH-DAY HOURS:MINUTES:SECONDS` |

## Message types
Messages can be one of three types
- `REQUEST`
- `RESPONSE`
- `FORWARD`

### Request messages
Requests are initiated by a client looking to communicate with another client and use the following header:
```
SMIFT/0.2 REQUEST <COMMAND>
```

Request messages **ALWAYS** have a `Timestamp` field in the body. This timestamp is used to store the time and date on which the request was originally generated and is of type `TIMESTAMP`. This is useful to ensure that message hashes are always different.

The full list of commands can be found in the [dedicated section](#Rquest-commands).

### Response messages
Response messages are either sent from the Routing Server to a requesting client, or from a handling client to the Routing Server. These messages use the following header:
```
SMIFT/0.2 RESPONSE <CODE> <PHRASE>
```

Response messages **ALWAYS** have a `Timestamp` field in the body. This timestamp is used to store the time and date on which the response was originally generated and is of type `TIMESTAMP`. This is useful for deferred responses.

Response codes and phrases can be found in the [dedicated section](#Response-messages).

### Forward messages
Forward messages are used by the SMIFT Routing Server to forward an incoming request to a handler node. Forward messages differ from request messages in commands and body contents, but use a similar header:
```
SMIFT/0.2 FORWARD <COMMAND>
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
| Message | `MESSAGE` | Sends a message to another node |
| FX publish | `FX PUBLISH` | Publishes an offer to the FX Bulletin Board |
| FX revoke | `FX REVOKE` | Revokes an offer published to the FX Bulletin Board |
| FX trade | `FX TRADE` | Issues an OTC order to purchase or liquidate currency |
| FX pull | `FX PULL` | Pulls one page of data from the FX Bulletin Board |

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
| `Destination-Account` | `SAID` | The payee's account identifier |
| `Source-Account` | `SAID` | The payer's account identifier |
| `Description` | `TEXT` | A note to help identify this transfer |

The server can reply with one **or more** of the following responses: `110`, `111`, `112`, `210`, `301`, `302`, `303`.

**NOTE:** `112`, and `210` responses may be deferred due to settlement and clearing delays.

### Cancel transfer
This command can only be issued by authenticated clients.
| Attribute | Type | Description |
| - | - | - |
| `Transaction-ID` | `STID` | The transfer to be cancelled |

The server can reply with one of the following responses: `113`, `211`, `301`, `302`, `303`.

**NOTE:** `211` responses may be deferred due to settlement and clearing delays.

### Message
This command can only be issued by authenticated clients.
| Attribute | Type | Description |
| - | - | - |
| `Destination-Server` | `SSID` | 
| `Content` | `TEXT` | The text to be sent |

The server can reply with one **or more** of the following responses: `120`, `220`, `301`, `302`, `303`.

### FX publish
This command can only be issued by authenticated clients.
| Attribute | Type | Description |
| - | - | - |
| `Currency` | `SCC`  | The currency for which the offer is being made |
| `Side` | `TEXT` | The side of the offer. Can be either `BUY` or `SELL` |
| `Exchange-Rate` | `DECIMAL` | The exchange rate beeing offered (expressed in the Designated Central Currency) |
| `Max-Amount` | `INT` | The maximum amount for which the offer is valid (expressed in the destination currency) |

The server can reply with one of the following responses: `221`, `301`, `302`, `303`.

### FX revoke
This command can only be issued by authenticated clients.

### FX trade
This command can only be issued by authenticated clients.

### FX pull
| Attribute | Type | Description |
| - | - | - |
| `Page-Index` | `INT` | The index of the desired page in the FX Bulletin Board. Used for larger networks |

The server can reply with one of the following responses: `222`, `301`, `302`, `303`.


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
| X2X | Communication and messaging |

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

#### 120 Message routed
This respose is sent after the SMIFT Routing Server has forwarded the message to the destination node.
| Attribute | Type | Description |
| - | - | - |
| `Request-Hash` | `SHA256` | The SHA256 hash of the initial request body |
| `Transaction-ID` | `STID` | The unique identifier used for the delivery of the message |

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

#### 220 Message delivered
This respose is sent after a message has been delivered to its destination node.
| Attribute | Type | Description |
| - | - | - |
| `Transaction-ID` | `STID` | The unique identifier used for the delivery of the message |

#### 221 FX offer published
This respose is sent by the SMIFT Routing Server after correctly processing an `FX PUBLISH` request.
| Attribute | Type | Description |
| - | - | - |
| `Request-Hash` | `SHA256` | The SHA256 hash of the initial request body |

#### 222 FX Bulletin Board retreived
This respose is sent by the SMIFT Routing Server as a reply to an `FX PULL` request and contains a maximum of 10 records.
| Attribute | Type | Description |
| - | - | - |
| `Request-Hash` | `SHA256` | The SHA256 hash of the initial request body |
| `Page-Index` | `INT` | The number identifier of the retreived page |
| `Retreived-Records` | `INT` | The number of retreived records |
| `Num-Pages` | `INT` | The total number of pages present on the FX Bulletin Board |

##### Payload
The payload contains a JSON array of market objects. Each market object is structured as follows:
| Attribute | SMIFT Type | JSON Type | Description |
| - | - | - | - |
| `currency` | `SCC` | `string` | The SCC of the currency for which the quotes are being displayed | 
| `officialBid` | `DECIMAL` | `float`, `null` | The bid price published by the currency's issuing server (if available) |
| `officialAsk` | `DECIMAL` | `float`, `null` | The ask price published by the currency's issuing server (if available) |
| `bestBid` | `DECIMAL` | `float`, `null` | The highest bid on the market (if available) |
| `bestAsk` | `DECIMAL` | `float`, `null` | The lowest ask on the market (if available) |
| `bestBidDepth` | `DECIMAL` | `float`, `null` | The amount of currency sellable at the highest bid price (if available) |
| `bestAskDepth` | `DECIMAL` | `float`, `null` | The amount of currency purchasable at the lowest ask price (if available) |
| `officialBidDepth` | `DECIMAL` | `float`, `null` | The amount of currency sellable at the official bid price (if available) |
| `officialAskDepth` | `DECIMAL` | `float`, `null` | The amount of currency purchasable at the official ask price (if available) |
| `totalBidDepth` | `DECIMAL` | `float`, `null` | The total amount of currency sellable (if available) |
| `totalAskDepth` | `DECIMAL` | `float`, `null` | The total amount of currency purchasable (if available) |
| `lastExecutionPrice` | `DECIMAL` | `float`, `null` | The exchange rate used for the last recorded transaction (if available) |
| `lastExecutionDate` | `DATE` | `string`, `null` | The date and time of the last recorded transaction (if available) |

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
This response is sent if one or more attributes of the initial request are invalid (e.g. uknown destination server, currency, account number, etc).

## Forward commands
Forward messages can be used to issue the following commands:
| Name | Code | Description |
| -    | -    | -           |
| Transfer | `TRANSFER` | Issues an order to transfer funds |
| Cancel transfer | `CANCEL TRANSFER` | Issues an order to cancel an unsettled transfer |
| Credit | `CREDIT` | Issues an order to credit funds to a user account |
| Message | `MESSAGE` | Delivers a message sent by another node |
| FX reserve | `FX RESERVE` | Reserves funds to be used for settlemnt of FX transactions |
| FX release | `FX RELEASE` | Releases funds recevied from (or reserved for) an FX transaction |

### Transfer
This command is issued by the SMIFT Routing Server and sent to a handling node for settlement.
| Attribute | Type | Description |
| - | - | - |
| `Transaction-ID` | `STID` | The ID associated by the SMIFT Routing Server with this transaction |
| `Amount` | `INT` | The amount to be transferred |
| `Currency` | `SCC` | The currency used for the transfer |
| `Destination-Server` | `SSID` | The payee's server |
| `Destination-Account` | `TEXT` | The payee's account identifier |
| `Source-Server` | `SSID` | The payer's server |
| `Source-Account` | `TEXT` | The payer's account identifier |
| `Settlement-System` | `TEXT` | The settlement system to be used for the transfer (can be: `Regular`, `SISM`, `FX`) |

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
| `Destination-Account` | `SAID` | The payee's account identifier |
| `Source-Server` | `SSID` | The payer's server |
| `Source-Account` | `SAID` | The payer's account identifier |
| `Description` | `TEXT` | A note to help identify this transfer |

### Message
This command is issued by the SMIFT Routing Server and sent to the final destination node to complete the delivery of the message.
| Attribute | Type | Description |
| - | - | - |
| `Transaction-ID` | `STID` | The ID associated by the SMIFT Routing Server with this transaction |
| `Sender-Server` | `SSID` | The sender server's identifier |
| `Content` | `TEXT` | The text 

## FX reserve
This message is sent by the SMIFT Routing Server to a handling node to reserve funds for the settlement of an FX transaction.

To be continued...

## FX release
This message is sent by the SMIFT Routing Server to a handling node to release funds received from (or reserved for) an FX transactin.

To be continued...

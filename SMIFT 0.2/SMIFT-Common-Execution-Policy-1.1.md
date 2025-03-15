The SMIFT Common Execution Policy, also known as _SCEP_, is a set of rules devised by SMIFT to ensure fault tollerance, transaprency and fairness in interserver transactions.

SCEP is heavily version dependent. Different SMIFTP versions may implement different SCEP revisions.

**NOTE:** Full SCEP compliance is **stricly required** in SMIFT Client/Server development.

## SMIFTP Reference
This revision of SCEP is relevant for SMIFTP version 0.2.

**NOTE:** This version of SCEPT has not been finalised yet.

## Glossary
| Term | Meaning |
| - | - |
| Server | Minecraft/Game Server|
| SMIFT Routing Server | The server software acting as SMIFT middleman |
| Transaction | Any SMIFT operation that needs to be forwarded to a third node, such as a transfer or message |
| Node | Programme used by Minecraft/Game server owners to interact with SMIFT |
| Client | Node that is requesting a service, such as a transfer |
| Execution Party | The institution responsible for settling a transfer |

## Response policy
The SMIFT Routing Server acts as a broker between nodes. Every time a node (e.g., Execution Party, Destination Node) issues a `RESPONSE` message to the SMIFT Routing Server as part of a transaction initiated by another node (known as a "client"), the latter is expected to forward the response to the client.

## Transfer issuance and settlement
### Message sequence
A transfer is always initiated by a client (called `Source`) by issuing a `TRANSFER` request and is intended to be received by another node (called `Destination`). The transfer is first sent to the SMIFT Routing Server that, after inspecting it, routes it to the correct Execution Party via a `TRANSFER` [forward](https://github.com/Alessandro-Salerno/SMIFT/wiki/SMIFT-Protocol-0.2#forward-messages) and replies with a `110` response. The Execution Party then checks if the origins's account balance is sufficient to cover the operation and, if so, debits the full amount to the origin, replying with a `111` response. 

The value of the `Settlement-System` attribute in the `TRANSFER` forward is chosen by the SMIFT Routing Server based on the following rules:
- If the `Destination-Account` attribute is not present or is not set **AND** the `Source-Account` attribute is not present or is not set, the attribute is set to `SISM`
- If the transfer is originating from an [FX trade request](https://github.com/Alessandro-Salerno/SMIFT/wiki/SMIFT-Protocol-0.2#fx-trade) instead of a transfer request, the attribute is set to `FX`
- In all other cases, the attribute i set to `Regular`

| Original request | `Destination-Account` attribute | `Source-Account` attribute | `Settlement-System` attribute |
| - | - | - | -|
| Transfer | Absent or not set | Absent or not set | `SISM` |
| Transfer | Set | Set | `Regular` |
| FX trade | | | `FX` |

The process then branches based on the content of the `Settlement-System` attribute in the `TRANSFER` forward received by the Execution Party.

#### Regular
A minimum grace period of **no less** than 24 hours is required before the Execution Party credits the full amount to the destination server and replies with a `112` response.

#### SISM
No grace period is required and the Execution Party is expected to credit the full amount to the destination server immediately. Upon settlement, the Execution Party replies with a `112` response.  Transfers using this settlement system are treated as inter-governmental transactions. As such, they're not credited to a private account on a third node, but rather to the destination server's account with the Execution Party. This procedure is part ofthe [SMIFT Insant Settlement Mechanism (SISM)](https://github.com/Alessandro-Salerno/SMIFT/wiki/SMIFT-Instant-Settlement-Mechanism-1.0).

#### FX
No grace period is required and the Execution Party is expected to credit the full amount to the destination server immediately. This settlement system implements a variant of SISM. Unlike other settlement systems, an Execution Party can authorise an FX transfer only if the [reserved balance](https://github.com/Alessandro-Salerno/SMIFT/wiki/SMIFT-Protocol-0.2#fx-reserve) of the source server is sufficient. Funds are also received by the destination server on its reserved balance.

| Settlement System | Grace period | Source of funds | Destination of funds |
| - | - | - | - |
| Regular | No less than 24 hours | Regular foreign deposit | Regular foreign deposit |
| SISM | None | Regular foreign deposit | Regular foreign deposit |
| FX| None | Reserved foreign deposit | Reserved foreign deposit |


Upon receiving a `112` response, the SMIFT Routing Server issues a `CREDIT` forward to the destination node that, in turn, **is expected** to credit the full amount to the final destination account (if specified) and reply with a `210` response.

#### Sequence diagram
1. Client issues `TRANSFER` request
2. SMIFT Routing Server routes the message to the correct Execution Party with a `TRANSFER` forward
3. SMIFT Routing Server replies to the client with a `110` response
4. Execution Party debits the full amount and replies with a `111` response (or a [300 series error]())
5. 24-hour grace period (if required by the settlement system)
6. Execution Party credits the full amount to the destination server's account and replies with a `112` reponse
7. SMIFT Routing Server sends a `CREDIT` forward to the destination node
8. Destinaton  node replies with a `210` response

### Execution Party routing
The Execution Party is chosen based on the currency used for a given transfer. Thus, the currency's official issuer is used as Execution Party and the transfer is likely to be routed to their node.

### Final destination policy
The destination node is in charge of crediting the transfer to the rightful recipiant. The final recipiant may, if so outlined by their server's [SMIFT Internal Execution Policy](https://github.com/Alessandro-Salerno/SMIFT/wiki/SMIFT-Internal-Execution-Policy), be credited in a different currency and might incur additional charges.

## Transfer cancellation
### Conditions
Transfers may only be cancelled **BEFORE** they're settled and credited to the final recipiant, i.e. **WITHIN** the 24-hour grace period. Cancellation past the 24-hour grace period may require direct consultation of the payee and Execution Party and is in **NO WAY** handled and regulated by SMIFT.

**NOTE:** SISM transfers are not subject to these terms. Cancellation of SISM transfers requires manual consultation of the parties involed and manual account reconciliation.

### Message sequence
A transfer cancellation order is always initiated by a client by issuing a `CANCEL TRANSFER` request, and intended to be routed to the transfer's original Execution Party. The order is first sent to the SMIFT Routing Server that, after inspecting it and checking the time delta, routes it to the appropriate Execution Party via a `CANCEL TRANSFER` [forward](https://github.com/Alessandro-Salerno/SMIFT/wiki/SMIFT-Protocol#forward-messages) and replies with a `113` response. Upon receiving the `CANCEL TRANSFER` forward, the Execution Party runs its own checks, credits the sum back to the original account, and replies with a `211` response.

#### Sequence diagram
1. Client issues `CANCEL TRANSFER` reuqest
2. SMIFT Routing Server checks the time delta and replies with a `113` response (or a [300 series error]())
3. SMIFT Routing Server routes the message to the original Execution Party via a `CANCEL TRANSFER` forward
4. Execution Party runs its own checks, nulls the transaction and replies with a `211` response (or a [300 series error]())

## Timings
When checking for time differences, such as in the case of time deadlines, nodes are **STRICLY FORBIDDEN** from using System now time and are instead required to use the message's `Timestamp` property.

## Deferred messages
If the end-point node for a given message is not availabe at the time of routing, the message will be cached by the SMIFT Routing Server and later sent to the intended node as soon as it connects. Deferred messages are delivered to nodes after they issue an `AUTHENTICATE` request and before a `200` response is sent. Deferred messages, however, are not sent if the authentication process fails with a `3XX` series response.
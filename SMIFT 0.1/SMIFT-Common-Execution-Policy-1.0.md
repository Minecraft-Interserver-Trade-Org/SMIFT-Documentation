The SMIFT Common Execution Policy, also known as _SCEP_, is a set of rules devised by SMIFT to ensure fault tollerance, transaprency and fairness in interserver transactions.

SCEP is heavily version dependent. Different SMIFTP versions may implement different SCEP revisions.

**NOTE:** Full SCEP compliance is **stricly required** in SMIFT Client/Server development.

## SMIFTP Reference
This revision of SCEP is relevant for SMIFTP version 0.1.

## Glossary
| Term | Meaning |
| - | - |
| Server | Minecraft/Game Server|
| SMIFT Routing Server | The server software acting as SMIFT middleman |
| Transaction | Any SMIFT operation that needs to be forwarded to a third node, such as a transfer or message |
| Node | Programme used by Minecraft/Game server owners to interact with SMIFT |
| Client | Node that is requesting a service, such as a transfer |
| Execution Party | The institution responsible for settling a transfer |

## Transfer issuance and settlement
### Message sequence
A transfer is always initiated by a client (called `Origin`) by issuing a `TRANSFER` request and is intended to be received by another node (called `Destination`). The transfer is first sent to the SMIFT Routing Server that, after inspecting it, routes it to the correct Execution Party via a `TRANSFER` [forward](https://github.com/Alessandro-Salerno/SMIFT/wiki/SMIFT-Protocol-0.1#forward-messages) and replies with a `110` response. The Execution Party then checks if the origins's account balance is sufficient to cover the operation and, if so, debits the full amount to the origin, replying with a `111` response. After **no less** than 24 hours, the Execution Party credits the full amount to the destination server and replies with a `112` response. Upon receiving it, the SMIFT Routing Server issues a `CREDIT` forward to the destination node that, in turn, **is expected** to credit the full amount to the final destination account and reply with a `210` response.

### Execution Party routing
The Execution Party is chosen based on the currency used for a given transfer. Thus, the currency's official issuer is used as Execution Party and the transfer is likelly to be routed to their node.

### Final destination policy
The destination node is in charge of crediting the transfer to the rightful recipiant. The final recipiant may, if so outlined by their server's [SMIFT Internal Execution Policy](https://github.com/Alessandro-Salerno/SMIFT/wiki/SMIFT-Internal-Execution-Policy), be credited in a different currency and might experience increases or reductions in the amount credited.

## Transfer cancellation
### Conditions
Transfers may only be cancelled **BEFORE** they're settled and credited to the final recipiant, i.e. **WITHIN** the 24-hour grace period. Cancellation past the 24-hour grace period may require direct consultation of the payee and Execution Party and are in **NO WAY** handled and regulated by SMIFT.

### Message sequence
A transfer cancellation order is always initiated by a client by issuing a `CANCEL ORDER` request, and intended to be routed to the transfer's original Execution Party. The order is first sent to the SMIFT Routing Server that, after inspecting it and checking the time delta, routes it to the appropriate Execution Party via a `CANCEL TRANSFER` [forward](https://github.com/Alessandro-Salerno/SMIFT/wiki/SMIFT-Protocol#forward-messages) and replies with a `113` response. Upon receiving the `CANCEL TRANSFER` forward, the Execution Party runs its own checks, credits the sum back to the original account, and replies with a `211` response.

## Timings
When checking for time differences, such as in the case of time deadlines, nodes are **STRICLY FORBIDDEN** from using System now time and are instead required to use the message's `Timestamp` property.

## Deferred messages
If the end-point node for a given message is not availabe at the time of routing, the message will be cached by the SMIFT Routing Server and later sent to the intended node as soon as it connects. Deferred messages are delivered to nodes after they issue an `AUTHENTICATE` request and before a `200` response is sent. Deferred messages, however, are not sent if the authentication process fails with a `3XX` series response.
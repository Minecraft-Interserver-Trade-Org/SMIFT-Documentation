The SMIFT Codes & Symbols Specification, or _SCSS_, is a set of standards that describe the form and type of codes and symbols used across all SMIFT networks.

## History
Development on this revision of the SMIFT Codes & Symbols Specification started on September 2nd, 2024.

## SMIFT Server Identifier (SSID)
The SMIFT Server Identifier is a unique eight-character-long alphanumeric code assigned to every Minecraft Server that joins a SMIFT Network.

SSIDs are **NOT** global i.e., different SMIFT networks may assign different SSIDs to the same Minecraft Server. 

Each SSID is comprised of three subcodes:
- Characters [0-3]: Abbreviated Minecraft Server name
- Characters [4-5]: [ISO 3166-2](https://en.wikipedia.org/wiki/ISO_3166-2) code of the real world country where the Minecraft Server was founded
- Characters [6-7]: Year in which the Minecraft Server was founded

### Full names
Full names for Minecraft Servers on a given SMIFT Network are unique. 

Full names cannot contain non-alphabetic characters (e.g., punctuation, numbers) and cannot be longer than 64 characters in total.

### Abbreviated names
The main candidate for the abbreviated name of a Minecraft Server on a SMIFT Network is derived by its full name using the method described here:
- Non-alphabetic characters in the full name are ignored. Spaces are used as delimiters for words, but are not allowed to be included in candidate names
- If the full name is exactly four letters long, the full name is used as candidate
- If the full name is shorter than four letters, the last letter is repeated until the length of the canidate name reaches four letters
- If the full name is comprised of a single word and is longer than four letters, the first four letters are used as candidate
- If the full name is comprised of multiple words, the first four uppercase initials are used to form the candidate. if there are less than four uppercase initials, letters from the right-most word are used to fill the remaining slots
- Trailing Xes are added if the candidate is shorter than four letters
- All characters are capitalised

The candidate is then assessed using the following steps:
- If the candidate is unique it is assigned to the Minecraft Server
- If the candidate has already been taken, characters are rearranged until they form an abbreviated name that is available.
- If all possible combinations of the four letters included in the candidate name are taken and the candidate contains trailing Xes that are not included in the full name, these are replaced from left to right until a unique candidate is found
- If all possible combinations of the characters included in the candidate and the ones used to replace the trailing Xes are taken,letters are replaced by similar-looking letters as follows. This step is iterative: one letter is replaced per iteration, starting from the left

| Letter A | Letter B |
| - | - |
| `U` | `V` |
| `U` | `W` |
| `V` | `W` |
| `I` | `Y` |
| `O` | `Q` |
| `P` | `R` |
| `M` | `W` |
| `N` | `M` |
| `C` | `G` |

- If none of the above steps result in a unique candidate, letters are replaced by similar-looking digits as follows. This step is iterative: one letter is replaced per iteration, starting from the right

| Letter | Number |
| - | - |
| `O` | `0` |
| `I` | `1` |
| `Z` | `2` |
| `E` | `3` |
| `A` | `4` |
| `S` | `5` |
| `G` | `6` |
| `Y` | `7` |
| `B` | `8` |
| `P` | `9` |

- If none of the above steps result in a unique candidate, but the names taken are from different real world countries and/or years, the candidate may be assigned anyway

- Finally, if none of the boave steps result in a unique candidate, the SMIFT Network's administration/software can either:

    A. Request the Minecraft Server administrators change the full name

    B. Generate a random sequence of letters and numbers until a unique name is found

### Examples
| Full name | Possible abbreviated name | Country of origin | Year founded | SSID |
| - | - |  - | - | - |
| Union of Minecraftian Socialist Republics | `UMSR`, `URSM`, `VMSR`, `VW5P`, ... | Italy | 2019 | `UMSRIT19`, ... |
| The CeglianCraft | `TCEG`, `TG36`, ... | Ireland | 2021  | `TCEGIE21`, ... |
| MSR | `MSRR`, `WZPR`, ... | United Kingdom | 2024 | `MSRRGB24`, ... |

This algorithm for generating abbreviated names guarantees the availability of an SSID for each Minecraft Server. 

## SMIFT Currency Code (SCC)
The SMIFT Currency Code is a unique nine-character-long alphanumeric code assigned to every currency used for inter-server financial transactions on a SMIFT Network.

The SCC for a given currency is obtained by appending a single letter to the SSID of the issuing Minecraft Server. The letter to be appended should be reflective of the currency's name.

### Currency Code Alias
Currency Code Aliases are used to avoid confusion and minimise mistakes. Aliases are three-character-long alphanumeric codes similar to [ISO 4217 codes](https://en.wikipedia.org/wiki/ISO_4217). 

The first two characters identify the issuing Minecraft Server and are assigned by the network administrators based on the server's SSID, while the third character is the same as the ninth in the original SCC. A theoretical maximum of 1296 Minecraft Servers can receive an alias on a given network.

Aliases should be preferred over SCCs whenever possible as they're easier to distinguish and remember.

## SMIFT Transaction Identifier (STID)
The SMIFT Transaction Identifier is a string of twentyeight decimal digits used to uniquely identify a SMIFT Transaction.

Each STID is comprised of four subcodes:
- Characters [0-3]: Year
- Characters [4-5]: Month
- Characters [6-7]: Day
- Characters [8-27]: Transaction number

### Transaction number
The transaction number is a sequential number used to identify a transaction relative to a day. This number is reset by the SMIFT Routing Server everyday at midnight.

### Examples
| Date | Time | SMIFT Transaction Identifier |
| - | - | - |
| 2024-09-09 | 17:30:00 | `2024090900000000000000000000` |
| 2024-09-09 | 17:30:01 | `2024090900000000000000000001` |
| 2024-09-09 | 17:30:02 | `2024090900000000000000000002` |
| 2024-09-10 | 15:20:00 | `2024091000000000000000000000` |
| 2024-09-10 | 15:20:01 | `2024091000000000000000000001` |

## SMIFT Account Identifier (SAID)
The SMIFT Account Identifier is a unique fourteen-character-long alphanumeric code assigned by servers to each account held by its own players, institutions, and organisations.

**NOTE:** Deposits held by other servers are identified by their SSID except in the case of deposits held at private depository institutions.

The SMIFT Account Identifier is used to identify source and destination accounts for transfers and other financial activities between entities over a SMIFT Network.

Each SAID is comprised of two subcodes:
- Characters [0-2]: Depository Institution Identifier
- Characters [3-13]: Account number

### Depository Institution Identifier
The Depository Institution identifier is a three-character-long alphabetic code used to uniquely identify a depository institution (such as a bank) on a given server. These codes are not global and are used coupled with SSIDs inside SMIFT Requests to correctly route the request.

Depository Institution Identifiers should resemble the name of the institution. For example, a hypotetical depository institution called BlockBank may be assigned `BBK`, `BLK` or `BLB` as Depository Institution Identifier.

### Account number
Account numbers are strings of eleven decimal digits used to uniquely identify an account held with a depository institution.

Examples of account numbers include: `77020113004`, `0000000001`, and `12345678901`.

**NOTE:** The account number `00000000000` is reserved for the depository institution itself.

### Examples
| Account holder | Depository Institution | SMIFT Account Identifier |
| - | - | - |
| Regular player | BlockBank | `BBK00000000123` |
| Regular player | EssentialsX Economy | `ESX00000000456` |
| Institution or organisation | Royal Creeper Bank | `RCB00000000789` |
| Other server | Central Bank / SMIFT Execution Party | |
| Other server | Import & Export Bank of Minecraftia | `IEB00000007777` |
| BlockBank | BlockBank | `BBK00000000000` |
| BlockBank | Royal Creeper Bank | `RCB00000008812` |

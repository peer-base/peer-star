# Decentralized Identity Systems Survey


|                                | uPort    | Sovrin   | Blockstack | ERC-725  |
| --------                       | -------- | -------- | ---------- | -------- |
| P2P Friendly                   | ❌       | ✔️        | ✔️         | ⚠️ <sup>[1]</sup>   |
| Verifiable Claims              | ⚠️ <sup>[2]</sup>   | ✔️      | ❌         | ⚠️ <sup>[3]</sup>   |
| Aliasing                       | ❌       | ✔️        | ✔️         | ❌       |
| Delegate Keys                  | ❌ <sup>[4]</sup>    | ✔️      | ❌         | ✔️       |
| Key Rotation & Revocation      | ✔️ <sup>[5]</sup>    | ✔️      |  ❌          | ✔️          |
| Social-Key Recovery            | ?         | ✔️      | ❌            | ✔️        |
| Costs                          | $        | ?         | $$ <sup>[6]</sup>        | $$$       |
| Identity Wallet UI             | ✔️        | ❌      | ✔️        | ❌ <sup>[7]</sup>   |

<sup>**[1]**</sup> There's no Identity Wallet applications that implement `erc725` yet.
<sup>**[2]**</sup> uPort claims & proofs is a simple key-value storage. Verifiable Claims can be stored within as values of that storage.
<sup>**[3]**</sup> `erc725` is studying the possibility to embrace Verifiable Claims but as of now, the `data` and `uri` of claims are agnostic.
<sup>**[4]**</sup> uPort is working on a new DID method, called `ethr`, that will support delegates: https://github.com/uport-project/ethr-did-resolver.
<sup>**[5]**</sup> Key rotation & revocation is possible via a (optional) Proxy Contract
<sup>**[6]**</sup> The identity creation is free but one must (optionally) pay ~0.001096 bitcoins to have a username
<sup>**[7]**</sup> OriginProtocol developed a "playground" to demonstrate the potential of the DID method, but it's not production ready nor enables users to authenticate to dApps: https://erc725.originprotocol.com

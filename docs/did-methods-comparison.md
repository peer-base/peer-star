# Decentralized Identity Systems Survey


|                                | uPort    | Sovrin   | Blockstack | ERC-725  |
| --------                       | -------- | -------- | ---------- | -------- |
| Identity Wallet UI             | ✔️        | ❌      | ✔️        | ❌ <sup>[1]</sup>   |
| P2P-friendly Authentication    | ❌       | ✔️        | ⚠️ <sup>[2]</sup>         | ⚠️ <sup>[3]</sup>   |
| Verifiable Claims              | ⚠️ <sup>[4]</sup>   | ✔️      | ❌         | ⚠️ <sup>[5]</sup>   |
| Aliasing                       | ❌       | ✔️        | ✔️         | ❌       |
| Delegate Keys                  | ❌ <sup>[6]</sup>    | ✔️      | ❌         | ✔️       |
| Key Rotation & Revocation      | ✔️ <sup>[7]</sup>    | ✔️      |  ❌          | ✔️          |
| Social-Key Recovery            | ?         | ✔️      | ❌            | ✔️        |
| Costs                          | $        | ?         | $$ <sup>[8]</sup>        | $$$       |

- <sup>**[1]**</sup> OriginProtocol developed a "playground" to demonstrate the potential of the DID method, but it's not production ready nor enables users to authenticate to dApps: https://erc725.originprotocol.com
- <sup>**[2]**</sup> It's unclear how one can prove control over the DID: https://forum.blockstack.org/t/got-a-few-questions-considering-using-blockstack/5608
- <sup>**[3]**</sup> There's no authentication libraries that implement `erc725` yet.
- <sup>**[4]**</sup> uPort claims & proofs is a simple key-value storage. Verifiable Claims can be stored within as values of that storage.
- <sup>**[5]**</sup> `erc725` is studying the possibility to embrace Verifiable Claims but as of now, the `data` and `uri` of claims are agnostic.
- <sup>**[6]**</sup> uPort is working on a new DID method, called `ethr`, that will support delegates: https://github.com/uport-project/ethr-did-resolver.
- <sup>**[7]**</sup> Key rotation & revocation is possible via a (optional) Proxy Contract
- <sup>**[9]**</sup> The identity creation is free but one must (optionally) pay ~0.001096 bitcoins to have a username

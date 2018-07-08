# Decentralized Identity Systems Survey


|                                | uPort    | Sovrin   | Blockstack | ERC-725  |
| --------                       | -------- | -------- | ---------- | -------- |
| Identity Wallet UI             | ✔️       | ❌ <sup>[1]</sup>       | ✔️        | ❌ <sup>[2]</sup>   |
| P2P-friendly Authentication    | ❌       | ❓ <sup>[1]</sup>       | ⚠️ <sup>[3]</sup>         | ❓ <sup>[4]</sup>   |
| Verifiable Claims              | ⚠️ <sup>[5]</sup>   | ✔️      | ⚠️ <sup>[6]</sup>        | ⚠️ <sup>[7]</sup>   |
| Aliasing                       | ❌       | ❌        | ✔️         | ❌       |
| Delegate Keys                  | ❌ <sup>[8]</sup>    | ✔️      | ❌         | ✔️       |
| Key Rotation & Revocation      | ✔️ <sup>[9]</sup>    | ✔️      |  ❌          | ✔️          |
| Social-Key Recovery            | ❓         | ✔️      | ❌            | ✔️        |
| Costs                          | $        | ❓         | $$ <sup>[10]</sup>        | $$$       |

- <sup>**[1]**</sup> Evernym is developing connect.me which will provide a Wallet for Sovrin.
- <sup>**[2]**</sup> [OriginProtocol](https://www.originprotocol.com/en) developed a "playground" to demonstrate the potential of the DID method, but it's not production ready nor enables users to authenticate to dApps: https://erc725.originprotocol.com.
- <sup>**[3]**</sup> It's unclear how one can prove control over the DID: https://forum.blockstack.org/t/got-a-few-questions-considering-using-blockstack/5608.
- <sup>**[4]**</sup> There's no authentication/client libraries that implement `erc725` yet.
- <sup>**[5]**</sup> uPort claims & proofs is a simple key-value storage. Verifiable Claims can be stored within as values of that storage.
- <sup>**[6]**</sup> We can prove ownership over social accounts but the data model is not compatible with Verifiable Claims. Nevertheless we probably could adapt data to be compatible.
- <sup>**[7]**</sup> `erc725` is studying the possibility to embrace Verifiable Claims but as of now, the `data` and `uri` of claims are agnostic.
- <sup>**[8]**</sup> uPort is working on a new DID method, called `ethr`, that will support delegates: https://github.com/uport-project/ethr-did-resolver.
- <sup>**[9]**</sup> Key rotation & revocation is possible via a (optional) Proxy Contract.
- <sup>**[10]**</sup> The identity creation is free but one must (optionally) pay ~0.001096 bitcoins to have a username.

# RFC Peer-Star-Identity

This is a proposal for an identity management system for Peer-Star, named Peer-Star-Identity. It's built upon standards to provide a good foundation for Peer-Star applications to authenticate and identify users.

*Authors*: André Cruz, João Santos, Pedro Teixeira

1. [Standards & foundations](#standards--foundations)
   1. [Decentralized Identifiers](#decentralized-identifiers-dids)
   1. [Self-signed Verifiable Claims](#self-signed-verifiable-claims)
   1. [Self-signed Key-chain](#self-signed-key-chain)
   1. [Trusting an entity](#trusting-an-entity)
1. [Proposal: The IdentityManager](#proposal-the-identitymanager)
   1. [What is it?](#what-is-it)
   1. [Managing identities](#managing-identities)
   1. [Managing Verifiable Claims](#managing-verifiable-claims)
   1. [Authenticating on applications](#authentication-on-applications)
   1. [Signing artifacts on applications](#signing-artifacts-on-applications)
   1. [Managing application sessions](#managing-application-sessions)
   1. [Revoking a device](#revoking-a-device)
1. [Diagrams & mockups](#diagrams--mockups)
1. [Glossary](#glossary)

## Standards & foundations

### Decentralized Identifiers (DIDs)

A Decentralized Identifier (DID) is a string that uniquely maps to an identiy, e.g. `did:btcr:123-445-333`. This standard is fully described in the [W3C DID spec](https://w3c-ccg.github.io/did-spec/) and is being widely adopted to create digital identities.

There are already several DID methods, such as uPort and Sovrin, which are listed in the [DID Method Registry](https://w3c-ccg.github.io/did-method-registry/). Each method has its own way to resolve a DID to a DID-Document. A DID-Document obeys a specified schema and it contains, among other things, a Public Key that entities can use to establish communication (encrypt, verify digital signatures) with the DID owner.

The owner of a DID can use the DID's private key to digitally sign artifacts. Any party can retrieve the DID-Document and verify the digital signature against the public key listed in the DID-Document.

### Self-signed Verifiable Claims

A Verifiable Claim is a qualification, achievement, quality, or piece of information about an entity's background such as a name, government ID, payment provider, home address, or university degree. This standard is described in detail in the [W3C Verifiable Claims spec](https://www.w3.org/TR/verifiable-claims-data-model).

By linking one or more Verifiable Claims to an identity, we are strengthening the credibility of the identity itself. Those claims may be self-signed or made by other entities.

We will start off by leveraging self-signed Verifiable Claims based on social networks, similar to [Keybase](https://keybase.io/) claims. They are easy to setup and they deliver a good base for trusting identities. Later on we can expand to other types of Verifiable Claims as well, like claims emitted by third-parties.

### Self-signed Key-chain

In a simple configuration, the DID key (the one used to prove control of the DID) could be used to sign artifacts and to cypher communication. However, that solution is sub-optimal for the following reasons:

- Does not offer Perfect Forward Secrecy: If an attacker manages to gain access to the private key, he/she is able to decypher all previous communications made to, and from, that entity's DID.
- Over exposes the private key: A private key should be used as few times as possible. Using it a lot makes it easier for an attacker to gain access to the key.
- If the key is compromised, the attacker is able to completely impersonate the real DID owner.

For those reasons, we propose a mechanism where a master key pair is used to sign (authorize) other key pairs. The master key pair can then be used to sign others. This process can continue through many layers. The advantage of this mechanism is that if a key is compromised, only the keys that it signed need to be changed. All the other ones above it, remain safe.

In the first version of this proposal we envision three layers of keys, as shown in the figure below. The first, *layer 1* is the DID key, called the *Root Key*. The Root Key is the one used to create the DID and authorize *layer 2* keys. Each entity will use Peer-Star applications from different devices (laptop, smart-phone and others). For each device, an entity will have one key pair whose public key is signed by the _Root Key_. Those keys are called Device Keys and are _layer 2_ keys. Finally, Device Keys can be used to sign (authorize) new temporary keys, Ephemeral Keys, to be used by specific applications. These are _layer 3_ keys.

```
(layer 1)  rootKey
(layer 2)   |-- deviceKey1 {signed by rootKey}
(layer 3)   |---- applicationKey1 {signed by deviceKey1}
(layer 3)   |---- applicationKey2 {signed by deviceKey1}
(layer 2)   |-- deviceKey_n_ {signed by rootKey}
```

#### Revocation

It's important for the entity in control of the DID to revoke a Device Key in case of theft or other circumstances. Similarly to how public keys are made available in DID-Documents, revoked Public Device Keys of a DID should also be public. The way a revoked Device Public Key gets published and the way a relying party query the list of revoked devices depends on the DID method as well.

#### Verifying the chain

A relying party must always verify the signatures of the Key-Chain to be sure that the entity controlling the DID authorized a specific public key. This is a three step process:

- Verify the Public Key signature of each layer, starting from the Leaf Key up to the Root Key.
- Check if the Root Public Key of the chain matches the DID Public Key.
- Check if the Device Public Key is not flagged as revoked.

At this point, a relying party just needs to prove that the entity owns the private key of the layer 3 key. This can easily be done by asking him to sign or decrypt a challenge.

### Trusting an entity

Typical DApps flows require a relying party to trust another. We can leverage DIDs, self-Signed Verifiable Claims and self-signed Key-chains in an "handshake" ceremony.

Consider the following example: Alice wants to share something secret with Bob.

After agreeing on a transport, Alice presents herself with her DID, her self-Signed Verifiable Claims and a self-signed Key-chain (linked list of public keys and signatures) to Bob. Alice does this by encrypting all this material, along with a nounce, with Bob's public key and signing it with her leaf private key (the last key on the chain). To trust Alice, Bob must:

1. Decrypt Alice's message and check its signature against the Leaf Public Key.
2. Resolve Alice's DID to a DID-Document which includes the Root Public Key.
3. Verify the Self-signed Key-chain as described in [Verifying the chain](#Verifying-the-chain)
4. Check the signatures of the self-signed Verifiable Claims against the Root Public Key.
5. Manually verify Alice's Verifiable Claims to see if they credible.
6. If Alice uses different keys for encrypting and signing, send her a challenge encrypted with her Public Key for encryption and ask her to send back the decrypted challenge.

Please note that certain aspects of point `5.` can be done automatically by crawling the proofs and verifying the signatures against Alice's Root Public Key. Still, Bob must explicitly verify the proofs as an attacker might be trying to impersonate the real Alice with fake social profiles.

If all went good, Bob is pretty confident that Alice is the real Alice. For future "handshake" ceremonies to be faster, Bob can store Alice's DID and Device Public Key somewhere, like a contacts list.

## Proposal: The IdentityManager

Usability is a crucial aspect on this proposal as it played a very important role on its concept.

On one hand, the experience for end-users should be intuitive, simple and familiar. Ideally, users should have little contact with private keys and most of the cryptographic operations should happen behind the scenes. On the other hand, developers building DApps should have an easy way to authenticate users.

### What is it?

The IdentityManager is a web-page hosted on IPFS and reachable via a specific URL, e.g.: https://peer-identity.io. The application will work completely offline thanks to the installation of a ServiceWorker.

Because it runs on its own domain, it provides a sandboxed environment where access to functionality and data is completely controlled. More specifically, all interactions made with the IdentityManager it will be made via [`postMessage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) where control and data segmentation is made via the messages' origin.

Using the `postMessage` API to directly communicate with the IdentityManager is a bit clunky. Instead, developers will use a library called IdentityManagerClient that abstracts all the communication layer with a intuitive API.

Moreover, any meaningful event on the IdentityManager, such as the removal of an identity, will be broadcasted to interested parties. This allows applications to immediately react to certain events by subscribing to them on the IdentityManager.

Among others, the most important advantages of this solution are:

- Doesn't require the Root Keys that control the DID to be used often, making them less vulnerable to theft as they can be kept safe.
- Embraces several DID methods instead of being tied to a specific method.
- Doesn't require any extensions, scanning of QRCodes, or any other unfriendly processes for daily use.
- Requires little updates to data stored in blockchains, hence incurring less costs.
- Acts like a "server" in the sense that sensitive information may be safely stored in it. Many DID methods require a secret for certain actions, e.g.: uPort requires an app secret to request normal and verified credentials.
- Has a wide support among devices. If we make it a PWA, users may install the application in the OS itself, further enhancing the user experience.
- Allows the user to choose between several identities when authenticating, useful for people that manage companies, organizations or similar.

### Managing identities

Users will be able to associate one or more identities using their preferred DID method. They will be guided throughout the process according to the chosen DID method. Once they complete it, they will have successfully created a Device Key associated with the identity Root Key (via signing).

All the Device Private Keys will be encrypted with a passphrase to improve security. Even if a device gets stolen, the robber won't be able to use most features without the passphrase as all the information stored locally will be encrypted with the Device Public Key. This gives time for the owner of the Identity to revoke the compromised device in the IdentityManager of another device.

Because the identity was linked to this device at setup time, and depending on the DID method, profile information and verifiable claims might become stale. For this reason, users will be able to sync up with their Root Identity to update any stale data.

### Managing Verifiable Claims

Users will be able to add claims to their identities. Initially, we will focus on adding self-signed claims linked to some of the most popular social networks. The process of adding those claims (and proofs) will work similarly to [Keybase](https://keybase.io/) in the sense that users will be guided throughout the process.

The exact way in how the claims will be stored is dependent on the DID method. For instance, uPort already provides a way to store these claims via attestations.

### Authentication on applications

Peer-Star applications will use the IdentityManagerClient to manage the user's session.

When bootstrapped, the IdentityManagerClient will read a pair of public and private keys, called the Session Keys, from the local storage. Alternatively, the Session Keys will be generated and stored in case they do not exist yet. Note that the Session Keys are considered layer 3 keys.
The IdentityManagerClient then takes the Session Public Key and queries the IdentityManager to retrieve the session data associated to it. If the IdentityManager responds back with the session data, the user is authenticated and, as such, there's an Authenticated Session. If that's not the case, the application can request the user to authenticate via the IdentityManagerClient.

When the application requests the IdentityManagerClient to authenticate the user, a popup pointing to the IdentityManager authenticate screen is open (via `window.open`). In this screen, the user is asked to choose an identity and to disclose some information, such as its name, its photo and a set of claims & proofs. If the user consented the disclosure of this information and entered the Device Private Key passphrase correctly, a new session for this application will be created and stored. The session data (including the signed Session Public Key) is sent back to the application. If the user denied the disclosure of the information, the operation will fail.
In both scenarios, the popup is closed and the outcome is made available to the application.

Applications might choose the TTL of a session depending on the degree of security they want to have.

### Signing artifacts on applications

Applications may want to sign artifacts, such as regular data or Ephemeral Keys. This will be possible in two different ways:

1. Sign with the Session Private Key
2. Sign with the Device Private Key

The first method is less intrusive and happens transparently to the user, but is less secure. More specifically, a robber who  stoled a device may still sign Ephemeral Keys with the Session Private Key as long as a session is valid (not expired). Relying parties will still see those signatures as valid until the DID owner revokes the stolen device.

The second method provides more security as the user is prompted for the Device Private Key passphrase, but it's more intrusive. The interaction is similar to the Authentication process, where a popup pointing to the sign screen gets open. The user either allows or denies the signing and the result gets back into the application. Please note that the passphrase might be stored in memory in the IdentityManager during a certain amount of time, which in turn makes this process less intrusive for subsequent signings.

Ultimately, the application may choose between both methods for different situations depending on the security degree they want to have.

### Managing application sessions

Users will be able to revoke any application session listed on the identity's Authenticated Session list. Revoking a session will essentially delete the application session from the IdentityManager local storage.

Even if a malicious application persists the session data for future use, it will be unable to sign artifacts, such as Ephemeral Keys, with the Device Private Key, rendering the application useless in those scenarios.

### Revoking a device

Users must be able to revoke a device associated with an identity. To do so, users may see the list of devices associated with an identity and revoke any device of that list, including the device from which they are interacting. This will require interacting with the DID method (Root Private Key) to publish the device being revoked.

If a device becomes aware that it was revoked, it will trigger a wipe-out process where all the identity data stored locally will be deleted, including the Device Key and Authenticated Sessions.

### Diagrams & mockups

**IdentityManager authentication flow diagram**

![Authentication flow](https://user-images.githubusercontent.com/1017236/41130726-db9125b4-6aaf-11e8-9ec1-bd55c16a7ff2.png)

**IdentityManager authenticate screen diagram**

![Authenticate screen](https://i.imgur.com/kyvkrZE.png)

**IdentityManager application mockups**

![IdentityManager authenticate screen mockup](https://i.imgur.com/fZ7VYfi.png)

## Glossary

- **Entity:** A person, company, organization or equivalent that controls an identity.
- **DID:** An identity identifier, which resolves to a DID-Document.
- **DID-Document:** A JSON-LD document which contains information about an identity, such as public keys to be used to communicate with entity that controls the identity.
- **Root Keys:** A pair of public and private keys that control the DID.
- **Device Keys:** A pair of public and private keys of a device, signed by the Root Public Key that controls the DID.
- **Ephemeral Keys:** A temporary pair of public and private keys, signed by the Device Private Key.
- **Session**: A unique pair of public and private keys stored locally by the IdentityManagerClient that identifies a visitor.
- **Authenticated Session:** A session that was signed by the entity controlling the Device Key.
- **IdentityManager**: An application that provides identification and authentication to the Peer-Star ecosystem.
- **IdentityManagerClient**: A library used by Peer-Star applications that makes it easier to interact with the IdentityManager.

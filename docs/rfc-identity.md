# RFC Peer-Star-Identity

This is a proposal for an identity management system for Peer-Star, named Peer-Star-Identity. It's built upon standards to provide a good foundation for Peer-Star applications to authenticate and identify users.

*Authors*: André Cruz, João Santos, Pedro Teixeira

1. [Standards & foundations](#standards--foundations)
   1. [Decentralized identifiers](#decentralized-identifiers-dids)
   1. [Self-signed Verifiable Claims](#self-signed-verifiable-claims)
   1. [DID Delegates & devices](#did-delegates--devices)
   1. [Using DID-Auth to prove control of the DID](#using-did-auth-to-prove-control-of-the-did)
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

### Decentralized identifiers (DIDs)

A Decentralized Identifier (DID) is a string that uniquely maps to an identity, e.g. `did:btcr:123-445-333`. This standard is fully described in the [W3C DID spec](https://w3c-ccg.github.io/did-spec/) and is being widely adopted to create digital identities.

There are already several DID methods, such as uPort and Sovrin, which are listed in the [DID Method Registry](https://w3c-ccg.github.io/did-method-registry/). Each method has its own way to resolve a DID to a DID-Document. A DID-Document obeys a specified schema and it contains, among other things, a set of public keys that entities can use to establish communication (encrypt, verify digital signatures) with the DID owner.

The owner of a DID can use the DID's private keys to digitally sign artifacts. Any party can retrieve the DID-Document and verify the digital signature against the public keys listed in the DID-Document.

### Self-signed Verifiable Claims

A Verifiable Claim is a qualification, achievement, quality, or piece of information about an entity's background such as a name, government ID, payment provider, home address, or university degree. This standard is described in detail in the [W3C Verifiable Claims spec](https://www.w3.org/TR/verifiable-claims-data-model).

By issuing Verifiable Claims about an entity's identity, we are strengthening the credibility of the identity itself. Those claims may be issued by the owners themselves or by other identities.

We will start off by leveraging self-issued and self-signed Verifiable Claims based on social networks, similar to [Keybase](https://keybase.io/) claims. They are easy to setup and they deliver a good base for trusting identities. Later on, we can expand to other types of Verifiable Claims as well, like claims emitted by third-parties.

### DID Delegates & devices

In a simple configuration, there's only a key that owns the DID, called the Master Key. That key could be used to sign artifacts and to cipher communication. However, that solution is sub-optimal for the following reasons:

1. Does not offer Perfect Forward Secrecy: If an attacker manages to gain access to the Master Key, he/she is able to decipher all previous communications made to, and from, that entity's DID. This happens because no session keys are generated, the Master key, acts as the session key.
2. Over exposes the private key: A Master Key should be used as few times as possible. Using it a lot makes it easier for an attacker to gain access to it.
3. If the Master Key is compromised, the attacker is able to completely impersonate the real DID owner.

The DID spec states that DID methods must provide a way for the owner to rotate keys, which solves `2`. The DID spec also states that DID methods must provide a way to recover the DID (e.g.: in case of theft), which solves `3`.

Nevertheless, the [DID specification](https://w3c-ccg.github.io/did-spec/) advises DID methods to support adding delegate keys that can act on behalf of the identity, but with granular capabilities. As an example, the [erc725](https://github.com/ethereum/EIPs/issues/725) DID method has such feature via adding keys with the `action` type, allowing such keys to perform signing or authentication.
delegate public keys will be listed in the DID-Document as well, which improves interoperability and compatibility with many other specs in the ecosystem, such as the [DID Auth](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/draft-documents/did_auth_draft.md) and [Identity Hubs](https://github.com/decentralized-identity/hubs/blob/master/explainer.md).

It's expected that the same entity will use Peer-Star applications from different devices, such as a desktop, laptop, smart-phone, or others. For the DID methods that support delegate keys, each device should have its own key added as a delegate. In case the DID method does not support delegate keys, the Master Key is used instead. In both cases, from now on, we will call these keys _Device Keys_.

#### Revocation

As previously stated, the entity in control of the DID may revoke a Device Key. Similarly to how non-revoked Public Device Keys keys are publicly listed in DID-Documents, revoked Public Device Keys should also be public. This is because a relying party must be able to verify signatures made in the past and, as such, must be able to assert that the public key is associated to the DID, even if it's revoked.

As of now, the DID spec does not specify how and where the revoked keys can be obtained. There's an [open issue](https://github.com/w3c-ccg/did-spec/issues/63) on the DID spec that brings this topic into the discussion and the W3C CCG working group is keen in listing revoked keys in the DID-Document.

### Using DID-Auth to prove control of the DID

Typical DApps flows require a relying party to trust another. We can leverage DIDs and self-signed Verifiable Claims in a "handshake" ceremony. The ceremony should follow the [DID-Auth spec](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/draft-documents/did_auth_draft.md).

To illustrate how [DID-Auth](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/draft-documents/did_auth_draft.md) works, consider the following example: Alice wants to share a secret with Bob.

After agreeing on a transport, Alice presents herself with a DID, a Device Public Key, and a set of self-signed Verifiable Claims. Alice does this by encrypting all this material, along with a nonce, with Bob's public key and signing her `authentication` key (a key present in the DID-Document under the `authentication` property). To trust Alice, Bob must:

1. Decrypt Alice's message
2. Resolve Alice's DID to a DID-Document
3. Check if Alice's Device Public Key is listed in `authentication` property of the DID-Document
4. Verify the message signature against her Device Public Key using the correct algorithm
5. Verify the signatures of the self-signed Verifiable Claims against the public keys listed in the `publicKey` property of the DID-Document.
6. Manually verify Alice's Verifiable Claims to see if they credible.

Please note that certain aspects of point `6.` can be done automatically by crawling the proofs and verifying the signatures against Alice's Public Keys. Still, Bob must explicitly verify the proofs as an attacker might be trying to impersonate the real Alice with fake social profiles.

If all went good, Bob is pretty confident that Alice is the real Alice. For future "handshake" ceremonies to be faster, Bob can store Alice's DID and Device Public Key somewhere, like a contacts list.

## Proposal: The IdentityManager

Usability is a crucial aspect of this proposal as it played a very important role in its concept.

On one hand, the experience for end-users should be intuitive, simple and familiar. Ideally, users should have little contact with private keys and most of the cryptographic operations should happen behind the scenes. On the other hand, developers building DApps should have an easy way to authenticate users.

The IdentityManager packages emerging [Standards & foundations](#standards--foundations) into an intuitive and easy to use interface for end-users.

### What is it?

The IdentityManager is a web-page hosted on IPFS and reachable via a specific URL, e.g.: https://peer-identity.io. The application will work completely offline thanks to the installation of a ServiceWorker. By using the application, users will be able to:

- Create identities on several DID methods
- Import identities created on other devices
- Manage Verifiable Claims of identities
- Authenticate to dApps

Because it runs on its own domain, it provides a sandboxed environment where access to functionality and data is completely controlled. More specifically, all interactions made with the IdentityManager it will be made via [`postMessage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) where control and data segmentation is made via the messages' origin. Using the `postMessage` API to directly communicate with the IdentityManager is a bit clunky. Instead, developers will use a library called IdentityManagerClient that abstracts all the communication layer with an intuitive API.

Moreover, any meaningful event on the IdentityManager, such as the removal of an identity, will be broadcasted to interested parties. This allows applications to immediately react to certain events by subscribing to them on the IdentityManager.

Among others, the most important advantages of this solution are:

- Embraces several DID methods instead of being tied to a specific method.
- Uses DID best practices, such as using delegate keys to associate devices.
- Doesn't require any extensions, scanning of QRCodes, or any other unfriendly processes for daily use.
- Acts like a "server" in the sense that sensitive information may be safely stored in it. Many DID methods require a secret for certain actions, e.g.: uPort requires an app secret to request normal and verified credentials.
- Has a wide support among devices. If we make it a PWA, users may install the application in the OS itself, further enhancing the user experience. Later on, we can develop native mobile apps to support OS's that don't yet allow PWA to be installed natively.
- Allows the user to choose between several identities when authenticating, useful for people that manage companies, organizations or similar.

### Managing identities

Users will be able to create identities using their preferred DID method or associate existing ones. They will be guided throughout the process according to the chosen DID method. Once they complete it, they will have successfully created a Device Key.

All the Device Private Keys will be encrypted with a passphrase to improve security. Even if a device gets stolen, the robber won't be able to use most features without the passphrase as all the information stored locally will be encrypted with the Device Public Key. This gives time for the owner of the Identity to revoke the compromised device in the IdentityManager of another device.

Because the identity was linked to this device at setup time, and depending on the DID method, profile information and verifiable claims might become stale. For this reason, users will be able to sync up with their identity to update any stale data.

### Managing Verifiable Claims

Users will be able to add claims to their identities. Initially, we will focus on adding self-signed claims linked to some of the most popular social networks. The process of adding those claims (and proofs) will work similarly to [Keybase](https://keybase.io/) in the sense that users will be guided throughout the process.

The exact way in how the claims will be stored is dependent on the DID method. For instance, uPort already provides a way to store these claims via attestations.

### Authentication on applications

Peer-Star applications will use the IdentityManagerClient to manage the user's session.

When bootstrapped, the IdentityManagerClient will attempt to read the Session Public Key from the local storage of the applications' origin. The IdentityManagerClient then takes the Session Public Key and queries the IdentityManager to retrieve the session data associated to it. If the IdentityManager responds back with the session data, the user is authenticated and, as such, there's an Authenticated Session. If the IdentityManager doesn't recognize that Session Public Key or if there's no Session Public Key in the first place, there's no Authenticated Session, meaning that no user is authenticated. In such cases, the application may request the Identity Manager to authenticate the user, typically via a login button.

When the application requests the IdentityManagerClient to authenticate the user, a popup pointing to the IdentityManager authenticate screen is open (via `window.open`). In this screen, the user is asked to choose an identity and to disclose some information, such as its name, its photo and a set of claims & proofs. If the user consented to the disclosure of this information and entered the Device Private Key passphrase correctly, a new session for this application will be created and stored. The session data, including the Session Public Key and its signature signed by the Device Private Key, is sent back to the application. If the user denied the disclosure of the information, the operation will fail.
In both scenarios, the popup is closed and the outcome is made available to the application.

Applications might choose the TTL of a session depending on the degree of security they want to have.

### Signing artifacts on applications

Applications may want to sign artifacts, such as regular data or Ephemeral Keys. This will be possible in two different ways:

1. Sign with the Session Private Key
2. Sign with the Device Private Key

The first method is less intrusive and happens transparently to the user, but is less secure. More specifically, a robber who stole a device may still sign Ephemeral Keys with the Session Private Key as long as a session is valid (not expired). Relying parties will still see those signatures as valid until the DID owner revokes the stolen device.

The second method provides more security as the user is prompted for the Device Private Key passphrase, but it's more intrusive. The interaction is similar to the Authentication process, where a popup pointing to the sign screen gets open. The user either allows or denies the signing and the result gets back into the application. Please note that the passphrase might be stored in memory in the IdentityManager during a certain amount of time, which in turn makes this process less intrusive for subsequent signings.

Ultimately, the application may choose between both methods for different situations depending on the security degree they want to have.

### Managing application sessions

Users will be able to revoke any application session listed on the identity's Authenticated Session list. Revoking a session will essentially delete the application session from the IdentityManager local storage.

Even if a malicious application persists the session data for future use, it will be unable to sign artifacts, such as Ephemeral Keys, with both the Session Key and the Device Key, rendering the application useless in those scenarios.

### Revoking a device

Users must be able to revoke a device associated with an identity. To do so, users may see the list of devices associated with an identity and revoke any device of that list, including the device from which they are interacting. This will require interacting with the DID method (Master Private Key) to publish the device being revoked.

If a device becomes aware that it was revoked, it will trigger a wipe-out process where all the identity data stored locally will be deleted, including the Device Key and Authenticated Sessions.

### Diagrams & mockups

**IdentityManager authentication flow diagram**

![Authentication flow](https://i.imgur.com/uMAi2be.png)

**IdentityManager authenticate screen diagram**

![Authenticate screen](https://i.imgur.com/kyvkrZE.png)

**IdentityManager application mockups**

![IdentityManager authenticate screen mockup](https://i.imgur.com/fZ7VYfi.png)

## Glossary

- **Entity:** A person, company, organization or equivalent that controls an identity.
- **DID:** An identity identifier, which resolves to a DID-Document.
- **DID-Document:** A JSON-LD document which contains information about an identity, such as public keys to be used to communicate with the entity that controls the identity.
- **Master Keys:** A pair of public and private keys that owns the identity (owns the DID).
- **Device Keys:** A pair of public and private keys of a device that can act on behalf of the identity owner.
- **Ephemeral Keys:** A temporary pair of public and private keys, signed by the Device Private Key or the Session Private Key.
- **Session**: A unique public key stored locally by the IdentityManagerClient that identifies a visitor.
- **Authenticated Session:** A Session that was signed by the entity controlling the Device Key. The Session Private Key lives securely on the IdentityManager local storage.
- **IdentityManager**: An application that provides identification and authentication to the Peer-Star ecosystem.
- **IdentityManagerClient**: A library used by Peer-Star applications that makes it easier to interact with the IdentityManager.

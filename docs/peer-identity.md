# Peer-Identity


## Index
* [Decentralized Identifier (DID)](#Decentralized-Identifier-DID)
* [Address Book](#Address-Book)
* [Peer-Pad Authentication & Read/Write Authorization](#Peer-Pad-Authentication-amp-ReadWrite-Authorization)
* [Key Recovery](#Key-recovery)
* [Resources](#Resources)

We start by describing DIDs. We then present the concept of Address Books. Finally we analyze how DIDs and Address Books can be used for providing access to peer-pad documents.

## Decentralized Identifier (DID)

Each entity is identified by a Decentralized Identifier (DID). A DID is an identifier, which obeys the schema DID:method:xxxx, where method indicates how that DID will resolve to an identity document (e.g. the uPort method is resolved by the uPort application, BTCR is resolved by a Bitcoin based application, and so on) and xxxx is the unique identifier of the individual.

When resolving a DID, a DID Document (DID-Doc) is returned. The DID-Doc contains useful information about an entity. This information contains the set of public keys meant to be used for establishing communication and proofs-of-personhood, such as a driver’s license, links to social network accounts, among other things.

![did-schema](https://github.com/joaosantos15/peer-star/blob/master/figures/did-diagram-v1.jpg?raw=true)

The entity in control of the identity has a pair of keys that should be stored in a secure endpoint. This can be viewed as the identity's root keys.

In the image below, Alice's root key is the same as her Ethereum account. The root public key is placed on Alice’s Ethereum Smart-Contract (in the image, Alice’s Smart-Contract), which is her DID-Document. Alice’s proofs of personhood are also maintained in the smart-contract (to reduce costs, the proofs can be pointed to using an IPFS link). Additionally, the root private key is stored and accessible in MetaMask.

**Key point:** At this point, anyone who verifies Alice’s proofs-of-personhood - that is, anyone who is convinced that Alice is indeed the owner of that smart-contract and has the identity she claims to have - now knows that PubRoot-Alice is Alice’s root public key.

Alice may wish to be able to authenticate herself using different devices. Currently, users of services such as Google Docs, are able to seamlessly use the services across all of their devices. The same should be possible with Peer-Identity. To achieve this, we propose a mechanism for device authorization depicted also in the image below. Looking at the case of authorizing a new smart-phone, she generates a new keypair for the device (the keys are generated locally in the device). Next, Alice uses her secure endpoint to sign the new public key, PubSmart-Phone-Alice using PrivRoot-Alice. The set of PubSmart-Phone-Alice and signature(PubSmart-Phone-Alice)PrivRoot-Alice is called the Presentation Package. Each device will have its own presentation package.

**Key point:** The presentation package is used to authenticate any device. Any entity can verify a presentation package, by checking the device key was signed by the user’s root key.


![presentation-package](https://github.com/joaosantos15/peer-star/blob/master/figures/presentation-package-v1.jpg?raw=true)



## Address Book

Each user has an address book. The address book contains information about known users. The Figure below shows how Alice presents herself to Bob, followed by Bob verifying Alice’s identity, and, finally, adding Alice to his address book.
Alice starts by sending Bob her presentation package(PP). She sends it from her smart-phone. Bob searches his Address Book for the DID provided by Alice , and concludes that he does not have that DID(step 1). Bob starts by resolving the DID, using a DID resolver (step 2). At this point, Bob can analyze Alice’s proofs of personhood, in order to verify that the entity who sent the PP is indeed Alice (step 3). After having verified Alice’s identity, Bob checks the digital signature of the PP, against Alice’s root public key(steps 4 and 5). After having verified the PP’s digital signature, Bob adds a new entry do his Address Book with Alice’s identification data. At this point, Bob can communicate with Alice by using the public key she provided.
Later, Alice sends a new PP, but this time, using her laptop. Bob verifies that he already has Alice’s DID in his address book. He verifies that the PP’s digital signature is authentic, and adds the new public key to Alice’s Authorized Keys list.

![image alt](https://github.com/joaosantos15/peer-star/blob/master/figures/address-book-v1.jpg?raw=true)

## Peer-Pad Authentication & Read/Write Authorization

In this section, we present how DIDs and Address book can be leveraged for authentication and access control in Peer-Pad. In order to access a Peer-Pad document, a user needs to have the document’s read and write keys. Each user with access to the document, can provide access to other users. This is done by sharing the read and/or write keys.

Each Peer-Pad document will have a guest list, a data-structure which contains the document’s read and write keys encrypted with the users public keys. 
The figure below illustrates the process of granting access to new users. 

In step 1, Alice requests access to the document from Bob. To do that, she sends him a Request for Access (RA), which is a message that contains her DID, a public key of a temporary key-pair she generated to be used only for the purposes of accessing this document, the document ID, and a digital signature of the message. This digital signature is signed by her authorized device’s private key (as described in Section Address Book).

In step 2, Bob verifies the request for access. He starts by checking whether Alice’s DID is in his contact book. In this example, she is. Next, Bob verifies if the key used to digitally sign the message is one of Alice’s authorized keys. To do that he checks his address book entry for Alice. If the verification is successful, Bob is assured that this message came from Alice. Next, in step 3, Bob generates an access key for Alice for document pp-doc-12345. This is done by encrypting the document’s keys [readKey,writeKey] with the temporary public key generated by Alice. Finally, Bob adds the access to the document’s guest-list. The guest list is a data structure kept in a CRDT embedded in the shared document (a CRDT is being used here to allow concurrent edits). 

Next, in step 4, Alice retrieves her authorization from the Guest-list. She decrypts the message using TemPrivKeyDoc-Alice, and extracts the documents read and write keys. The TemPrivKeyDoc-Alice is generated using a user defined passphrase as a seed. This means that the user can reconstruct the private key, regardless of the device she is in. In this example, Alice starts by requesting access to the document from her smart-phone. Eventually, Bob adds Alice to the guest list for that document. Later, Alice may be using her laptop. She can access the document Guest-list, and recover the private key to decrypt the document’s Keys, by entering her passphrase.
Finally, in step 5, she views and edits the document.
It is important to note that these steps need not to occur in sync. Steps 2 and 3 can happen while Alice is offline. When Alice is back online, steps 4 and 5 can take place, even if Bob if offline.

![peer-pad-access](https://github.com/joaosantos15/peer-star/blob/master/figures/peer-pad-access-v1.jpg?raw=true)





## Key recovery
(Taken from DKPI v1.0.0)

### Master key

>Recovering from master key loss
- Recombining shards of the master key: using something like "Shamir Secret Sharing" or "Threshold signatures", we can recreate a lost master key.

>Protecting against compromise
- The danger of compromise comes from a single identity having a master key in their possession any point in time. We can address this issue by ensuring that no single entity possesses the master key at any time.
To mitigate this, when generating a master key, generate it ephemerally, and then break it into shards and store these shards separately.

>Using Smart Contracts
- Principals can create recovery mechanisms in smart contracts. For instance, a smart contract can be coded to function only when it receives a message signed by 6 out of 10 entities, or follow any other arbitrary logic.

### Subkeys (recovery or revocation)
- Here we are using sub-keys to use in the devices, which is what we use to interact with third-parties.
- Subkey compromise or loss is less of a concern than loss or compromise of a master key. If a subkey is loss or compromises the master key can be used to securely generate and replace old keys. However, depending on how they're used, old subkeys might still require revocation.
- Revoking a subkey can be done by sending a message to all the peers we have interacted with, signed by the master key.

## Comparison to other solutions (under development)
- **Sovrin** 
    - Unlike Sovrin, this solution does not allow selective disclosure. In this solution, by providing a DID, we're disclosing our entire identity to any one that can peek into the content of the Ethereum blockchain.
    - In that line, Sovrin uses pseudonym DIDs when interacting with a verifier. Instead, we use a unique DID, which we leak inside our presentation package. This exposes our unique DID as a target.
    - Using Sovrin, we can present Zero-knowledge proofs (ZKPs) to verifiers. For instance, a person could disclose the proof that they are at least 18 years old without disclosing their actual age.
- **uPort**
    - https://developer.uport.me/overview 

- **IPID**
    - Based on IPNS, does not require a global consensual block-chain

- **Keybase**



- **DID-Auth**
    - A protocol for a user proofs he is in control of a DID.
    - The entities agree on a transport protocol for exchanging authentication messages.
    - The transport protocol an entity wishes to use, may be described in its DID-Doc. E.g. “Hi, I am Bob, this is my DID-Doc, and I’d like you to use this server, https://BobsAuthServer.com to perform DID authentication”
    - The entity verifying the DID is called relying party. The entity being authenticated is called identity owner.
    - The relying party sends a challenge to the identity owner. The identity owner signs the response using a key pair listed in its DID-Document.

## Resources
- https://docs.google.com/document/d/1s1f6b_aP1-sThWZjjeQQSHvZq8rvjiuDgoNY3-UJaso/edit?usp=sharing 
- https://hackmd.io/nvpIVhr7Q0KIR5erICWwtQ 
- https://github.com/decentralized-identity/universal-resolver 
- https://github.com/ipfs-shipyard/peer-star/issues/3#issuecomment-387892432 
- https://github.com/ipfs-shipyard/dapp-identity-api 
- https://github.com/protocol/pl-t-andyet/issues/44 
- https://github.com/ipfs-shipyard/peer-star/issues/3 
- https://hackmd.io/s/H17TwVqsz# 
- https://github.com/ipfs/dynamic-data-and-capabilities/issues/4 
- https://github.com/ipfs/dynamic-data-and-capabilities/issues/15 
- https://github.com/ipfs/dynamic-data-and-capabilities/issues/12 
- https://github.com/ipfs/dynamic-data-and-capabilities/issues/7 
- https://github.com/ipfs-shipyard/peer-star/issues/6 
- https://github.com/ipfs-shipyard/peer-star/issues/7 
- DID-Recovery: https://github.com/decentralized-identity/did-recovery 
- Side-Tree Entity Protocol: https://github.com/decentralized-identity/did-methods/blob/master/sidetrees/explainer.md 

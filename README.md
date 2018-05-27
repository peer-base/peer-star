# peer-star

Distributed Applications and their internal building blocks exposed as reusable components that can be used for all kinds of p2p usecases

[`peer-star`](https://github.com/search?q=topic%3Apeer-star+org%3Aipfs-shipyard+fork%3Atrue)

## Applications

- [peer-pad](https://github.com/ipfs-shipyard/peer-pad)
- [peer-flipchart](https://github.com/ipfs-shipyard/peer-flipchart)
- [peer-blog](https://github.com/ipfs-shipyard/peer-blog)

## Building blocks

- [peer-pad-core](https://github.com/ipfs-shipyard/peer-pad-core)
- [peer-crdt](https://github.com/ipfs-shipyard/peer-crdt)
- [peer-crdt-ipfs](https://github.com/ipfs-shipyard/peer-crdt-ipfs)
- [peer-crdt-bind-codemirror](https://github.com/ipfs-shipyard/peer-crdt-bind-codemirror)

## Examples

- [peer-crdt-example](https://github.com/ipfs-shipyard/peer-crdt-example)

## API

### PeerStar

```js
const PeerStar = require('peer-star')
```

### Self-Identity

```js
const identity:Identity = PeerStar.identity([identityStore:IdentityStore])
const identities:Map<String:Identity> = identity.all()
const identity:Identity = identity.get('identity-id')
```

### Collaboration

```js
const app:App = await PeerStar.app('my app id')
const collaboration:Collaboration = await app.collaborate('room name', identity)
```

### Document

```js
collaboration.on('new document', (docName:String, whoDidIt:Peer) => {
  console.log('document was created', docName)
})

collaboration.on('document removed', (docName:String, byPeer:Peer) => {
  console.log('document %s was removed by peer', docName, byPeer)
})

collaboration.on('peer wants access', (peer:Peer, docName:String, accessLevelRequested:String) => {
  await collaboration.grantAccess(peer, docName, accessLevelRequested)
  await collaboration.revokeAccess(peer, docName, 'write')
})

// get doc
const doc:Doc = await collaboration.getDocument('document name')

// request access to doc
const permission:DocPermission = await doc.requestAccess('write')

// create doc
const doc2 = await collaboration.create('document type')

doc2.name // has the name of the document

doc.on('changed', (what:ChangeEvent, who:Peer) => {
  // document changed
})

doc.on('stable', (lastChange:ChangeEvent) => {
  // doc latest change is causally stable
})

doc.on('replicated', (peer:Peer) => {
  // doc latest changes are replicated to given peer
})

doc. // mutate and access doc

// peers
const peers:Set<Peer> = doc.peers()

// history of changes, paginated
const history:Sequence<ChangeEvent> = doc.history([from [, count]])

await doc.leave() // leave document

await collaboration.leave()
```

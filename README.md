# bittorrent-wire [![travis][travis-image]][travis-url] [![npm][npm-image]][npm-url] [![downloads][downloads-image]][downloads-url] [![Greenkeeper badge](https://badges.greenkeeper.io/CraigglesO/bittorrent-wire.svg)](https://greenkeeper.io/)

[travis-image]: https://travis-ci.org/CraigglesO/bittorrent-wire.svg?branch=master
[travis-url]: https://travis-ci.org/CraigglesO/bittorrent-wire
[npm-image]: https://img.shields.io/npm/v/bittorrent-wire.svg
[npm-url]: https://npmjs.org/package/bittorrent-wire
[downloads-image]: https://img.shields.io/npm/dm/bittorrent-wire.svg
[downloads-url]: https://npmjs.org/package/bittorrent-wire

### A stream ready wire for the Bittorrent Protocol

One of the fastest, lightest, and smartest bittorrent wires yet.

| streams | protocol |
|---|---|
| ![stream-duplex](https://github.com/CraigglesO/bittorrent-wire/blob/master/stream-duplex.png) | ![stream-duplex](https://github.com/CraigglesO/bittorrent-wire/blob/master/bittorrent-swarm.png)

#### Bittorrent Protocol
  * [WIKIPEDIA](https://wiki.theory.org/BitTorrentSpecification#Handshake)
  * [BEP_0003](http://www.bittorrent.org/beps/bep_0003.html)

#### Streams
* [THE STREAM HANDBOOK](https://github.com/substack/stream-handbook)
* [Duplex Streams](https://nodejs.org/api/stream.html#stream_duplex_and_transform_streams)


#### Extension Protocol
| Topic | Source |
|---|---|
| Extension | [BEP_0010](http://www.bittorrent.org/beps/bep_0010.html) |
| UT_PEX | [BEP_0011](http://www.bittorrent.org/beps/bep_0011.html) |
| UT_METADATA | [BEP_0009](http://www.bittorrent.org/beps/bep_0009.html) |

| module | tests | version | description |
|---|---|---|---|
| [ut-extensions][ut-extensions] | [![][ut-extensions-ti]][ut-extensions-tu] | [![][ut-extensions-ni]][ut-extensions-nu] | Extensions for The Bittorent Protocol

[ut-extensions]:    https://github.com/CraigglesO/ut-extensions
[ut-extensions-ti]: https://travis-ci.org/CraigglesO/ut-extensions.svg?branch=master
[ut-extensions-tu]: https://travis-ci.org/CraigglesO/ut-extensions
[ut-extensions-ni]: https://img.shields.io/npm/v/ut-extensions.svg
[ut-extensions-nu]: https://npmjs.org/package/ut-extensions


## Install

``` javascript
npm install bittorrent-wire
```

## Usage

**Basic**
``` javascript
import Wire from "bittorrent-wire"

let wire = new Wire("INFO_HASH_GOES_HERE", "PEER_ID_GOES_HERE");

wire.pipe(wire);
```

**Online**
``` javascript
import * as net from 'net';
import Wire     from "bittorrent-wire"

let socket = net.connect(1337, 'localhost');

socket.once('connect', () => {
  let wire = new Wire("e940a7a57294e4c98f62514b32611e38181b6cae", "-EM0022-PEANUTS4AITH");
  socket.pipe(wire).pipe(socket);

  wire.on("handshake", (infoHash, peerID) => {
    // ...
  });
});

```

### Function-Event Pair

#### Outbound [Handshake] ![#c5f015](https://placehold.it/15/c5f015/000000?text=+)
``` javascript
wire.sendHandshake();
```

#### Inbound [Handshake] ![#1589F0](https://placehold.it/15/1589F0/000000?text=+)
``` javascript
wire.on("handshake", function (infoHash: Buffer, peerId: Buffer) {
  // infoHash -> 20 bit hex buffer
  // peerId   -> 20 bit utf-8 buffer
});
```

#### Outbound [Not Interested] ![#c5f015](https://placehold.it/15/c5f015/000000?text=+)
``` javascript
wire.sendHandshake();
wire.sendNotInterested();
```

#### Inbound [Not Interested] ![#1589F0](https://placehold.it/15/1589F0/000000?text=+)
``` javascript
wire.on("close", () => {
  // wire.isActive -> false
});
```

#### Outbound [Interested] ![#c5f015](https://placehold.it/15/c5f015/000000?text=+)
``` javascript
wire.sendHandshake();
wire.sendInterested();
```

#### Inbound [Interested] ![#1589F0](https://placehold.it/15/1589F0/000000?text=+)
``` javascript
wire.on("interested", () => {
  // wire.choked -> false
});
```

#### Outbound [Have] ![#c5f015](https://placehold.it/15/c5f015/000000?text=+)
``` javascript
wire.sendHandshake();
wire.sendHave(1); //Index number
```

#### Inbound [Have] ![#1589F0](https://placehold.it/15/1589F0/000000?text=+)
``` javascript
wire.on("have", (index: number) => {
  // index -> 1
});
```

#### Outbound [Bitfield] ![#c5f015](https://placehold.it/15/c5f015/000000?text=+)
``` javascript
let buffer = Buffer.from("40", "hex");
wire.sendHandshake();
wire.sendBitfield(buffer); // Hex Buffer
```

#### Inbound [Bitfield] ![#1589F0](https://placehold.it/15/1589F0/000000?text=+)
``` javascript
wire.on("bitfield", (bits) => {
  // bits.toString("hex") -> 40
});
```

#### Outbound [Request] ![#c5f015](https://placehold.it/15/c5f015/000000?text=+)
``` javascript
const TPH = require("torrent-piece-handler");
const files = [ { path: "Downloads/lol1/1.png",
         name: "1.png",
         length: 255622,
         offset: 0 },
       { path: "Downloads/lol2/2.png",
         name: "2.png",
         length: 1115627,
         offset: 255622 } ];
// ...

wire.sendHandshake();
wire.sendInterested();
// Prepare a request for the pieces
let tph = new TPH.default(files, 962416635, 1048576, 918, 872443);

tph.prepareRequest(0, (buf, count) => {
  // Send to wire.
  wire.sendRequest(buf, count);
});
```

#### Inbound [Request] ![#1589F0](https://placehold.it/15/1589F0/000000?text=+)
``` javascript
wire.on("request", () => {
  // wire.inRequests[2].index  -> 0
  // wire.inRequests[2].begin  -> 16384 * 2
  // wire.inRequests[2].length -> 16384
});
```

#### Outbound [Cancel] ![#c5f015](https://placehold.it/15/c5f015/000000?text=+)
``` javascript
wire.sendHandshake();
wire.sendInterested();

wire.sendCancel(0, 16384 * 2, 16384); //Index number
```


#### Inbound [Cancel] ![#1589F0](https://placehold.it/15/1589F0/000000?text=+)
``` javascript
wire.on("cancel", (index, begin, length) => {
  // index  -> 0
  // begin  -> 16384 * 2
  // length -> 16384
});
```

## API

### Class: Wire

``` javascript
new Wire(infoHash: string | Buffer, myID: string | Buffer, options?: Options)
```
* **infoHash**: the torrent info_hash
  * 20 byte hex Buffer or 20 character string
  * ex: `"e940a7a57294e4c98f62514b32611e38181b6cae"`
* **myID**: the ID of the user.
  * 20 byte hex Buffer or 20 character string
  * ex: `"-EM0022-PEANUTS4AITH"`
* Options:
``` javascript
interface Options {
  "metadata_handshake": MetadataHandshake;
}

interface MetadataHandshake {
  "ipv4":          Buffer;
  "ipv6":          Buffer;
  "m": Extension;
  "metadata_size": number;
  "p":             number;
  "reqq":          number;
  "v":             Buffer;
  "yourip":        Buffer;
}

interface Extension {
  "ut_pex":      number;
  "ut_metadata": number;
}
```

### Wire: Outbound

#### Wire.sendHandshake() `<pstrlen><pstr><reserved><info_hash><peer_id>`
The first step in starting a channel with another peer

#### Wire.sendInterested() `<len=0001><id=2>`
This will request unchoking the connection

#### Wire.sendNotInterested() `<len=0001><id=3>`
If the peer does not have data we need

#### Wire.sendHave(index: number) `<len=0005><id=4><piece index>`
* `index` the block in question [e.g. 0, 1, 2, 3, ...]
* Send peers that you have a new piece in your bitfield

#### Wire.sendBitfield(bitfield: string) `<len=0001+X><id=5><bitfield>`
* `bitfield` string representing a hex bitfield [e.g. "f8"] [binary: 1111 1000]
* Send peer your current downloaded bitfield

#### Wire.sendRequest(payload: Buffer, count: number) `<len=0013><id=6><index><begin><length>`
* `payload` wrap the index, begin, and length all into one buffer
* `count` number of pieces you are requesting [e.g. 0, 1, 2, 3, ...]
* Send a request for either a piece or block. Payload allows for multiple requests

#### Wire.sendPiece(piece: Buffer) `<len=0009+X><id=7><index><begin><block>`
* `piece` wrap the index, begin, and length all into one buffer
* Given a request, this is how you respond

#### Wire.sendCancel(index: number, begin: number, length: number) `<len=0013><id=8><index><begin><length>`
* `index` index of the block [e.g. 0, 1, 2, 3, ...]
* `begin` number inside the block [e.g. 0 * 16384, 1 * 16384, 2 * 16384, ...]
* length: length (always 16384 unless last piece)
* Canceling a download with the peer

#### Wire.sendPort(port: number) `<len=0003><id=9><listen-port>`
* `port` number of the UDP port the DHT is listening on
* send peer your KPRC protocol UDP port

### Wire: Extensions

#### Wire.sendMetaHandshake()
* if *"metadata_handshake"* options are set, this will send your options specified, otherwise the defaults are:
  * `ut_metadata` 2
  * `ut_pex` 1

#### Wire.createExtensions(metadataSize: number, torrentInfo: Info)
  * `this.createUTmetadata(metadataSize, torrentInfo)`
  * `this.createUTpex()`

#### Wire.createUTmetadata(metadataSize: number, torrentInfo: Info)
  * `this.ext[UT_METADATA] = new UTmetadata(metadataSize, this.infoHash, torrentInfo)`

#### Wire.createUTpex()
  * `this.ext[UT_PEX] = new UTpex()`

#### Wire.prototype.sendPexPeers (addPeers: Array<string>, addPeers6: Array<string>, dropPeers: Array<string>, dropPeers6: Array<string>)
  * `addPeers` ipv4 peers that you know are good
  * `addPeers6` ipv6 peers that you know are good
  * `dropPeers` ipv4 peers that you know are bad
  * `dropPeers6` ipv6 peers that you know are bad

### Wire: High Level Calls

#### Wire.isChoked(): Boolean
  * `return this.choked`

#### Wire.isBusy(): Boolean
  * `return this.busy`

#### Wire.setBusy()
  * `this.busy = true`

#### Wire.unsetBusy()
  * `this.busy = false`

#### Wire.closeConnection()
  * `this.isActive = false`
  * `this.emit("close")`

#### Wire.removeMeta()
  * `this.meta = false`
  * `delete this.ext[ UT_METADATA ]`

#### Wire.removePex()
  * `delete this.ext[ UT_PEX ]`

## Debug
`DEBUG="bittorrent-wire" ts-node bittorrent-wire.ts`

## ISC License (Open Source Initiative)

ISC License (ISC)
Copyright 2017 <CraigglesO>
Copyright (c) 2004-2010 by Internet Systems Consortium, Inc. ("ISC")
Copyright (c) 1995-2003 by Internet Software Consortium


Permission to use, copy, modify, and/or distribute this software for any purpose with or without fee is hereby granted, provided that the above copyright notice and this permission notice appear in all copies.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.

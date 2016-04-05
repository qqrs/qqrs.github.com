---
layout: post
title: "Writing a BitTorrent Client"
date: 2016-04-04 19:21
comments: false
categories: code
---

Writing a BitTorrent client has been interesting side project to learn concurrent networking concepts and practice structuring a new project from scratch.

However, when I first started I had one important question: "Do I actually want to write a BitTorrent client?"

I found that I had to do much more reading than I expected to understand the BitTorrent protocol and to decide whether the scope was appropriate for a spare time project.

As a result, this post is a very short overview of a minimal BitTorrent download implementation.

## 1. Parse the `.torrent` metainfo file
The `.torrent` file contains information about the torrent tracker and the files to be downloaded.
Data is encoded using a serialization protocol called bencoding.
If your language provides a bencoding library, this is not much more difficult to decode than parsing json.

## 2. Connect to the tracker
To connect to the torrent, an HTTP GET request is made to the tracker announce URL.
The response provides a list of available peers.

## 3. Concurrent peer network connections
The client will connect to peers using TCP sockets.
To support multiple simultaneous connections the client should be able to handle network operations asynchronously.
There are two fundamental ways to do this: (1) using threads, or
(2) using an event loop with select() (or a library like Twisted which does so internally).

## 4. Peer protocol
The spec defines a number of messages that each peer must be prepared to send and receive.
A minimal download client may not need to implement all of these messages.
In order to start downloading from a peer, a client needs to send a handshake, wait for a handshake response,
send an 'interested' message, and wait for an 'unchoke' message.
It can then start sending 'request' messages to request blocks.
The peer will respond with 'piece' messages which contain the block data.

## 5. Torrent strategy
The client must download all blocks of all pieces and assemble them into the complete output file
set. If any peers disconnect or fail to provide a block, the client must request from another peer. A more ambitious client may also attempt to further optimize its download strategy to improve download times.

## Further reading

These posts each go into more detail regarding implementation details:

[How to Write a Bittorrent Client (part 1)](http://www.kristenwidman.com/blog/33/how-to-write-a-bittorrent-client-part-1/) 
[(part 2)](http://www.kristenwidman.com/blog/71/how-to-write-a-bittorrent-client-part-2/)
(Kristen Widman)  
[Pitfalls when creating a BitTorrent client](http://charmeleon.github.io/advice/2012/11/26/pitfalls-when-creating-a-bittorrent-client/) (Erick Rivas)

The best advice I picked up from them is to rely on the [unofficial BitTorrent spec](https://wiki.theory.org/BitTorrentSpecification) and to use Wireshark liberally to inspect the network traffic.

Since there now many extensions to the BitTorrent protocol, you should test with torrents that do not use new or experimental features. I have tested with torrents from [archive.org](https://archive.org/details/bittorrent) and [bt.etree.org](http://bt.etree.org/).

Good luck!

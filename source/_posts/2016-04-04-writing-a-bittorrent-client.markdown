---
layout: post
title: "Writing a BitTorrent Client"
date: 2016-04-04 19:21
comments: false
categories: code
---

I've found that writing a BitTorrent client from scratch has been an interesting project for practicing concurrent networking concepts.
However, when I started, I found that I had to do much more reading than I expected to understand the BitTorrent protocol and to decide whether the scope was appropriate for a spare time project.

As a result, this post is a short overview of a minimal BitTorrent download implementation.

I hope it will be useful to other programmers seeking an answer to the question: "Do I actually want to write a BitTorrent client?"

## 1. Parse the `.torrent` metainfo file
The `.torrent` file contains information about the torrent tracker and the files to be downloaded.
Data is encoded using a serialization protocol called bencoding.
Parsing bencoded data is not significantly more difficult than parsing json, and there is likely a bencoding library available for your language.

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
set. If any peers disconnect or fail to provide a block, the client must request from another peer.
A more ambitious client may also attempt to further optimize its download strategy to improve download times.

## Further reading

I found the following blog posts to be very helpful when I was getting started:

[How to Write a Bittorrent Client (part 1)](http://www.kristenwidman.com/blog/33/how-to-write-a-bittorrent-client-part-1/) 
[(part 2)](http://www.kristenwidman.com/blog/71/how-to-write-a-bittorrent-client-part-2/)
(Kristen Widman)  
[Pitfalls when creating a BitTorrent client](http://charmeleon.github.io/advice/2012/11/26/pitfalls-when-creating-a-bittorrent-client/) (Erick Rivas)

The best advice I picked up from them is (1) to rely on the [unofficial BitTorrent spec](https://wiki.theory.org/BitTorrentSpecification), and (2) to use Wireshark to inspect network traffic to clarify ambiguities in the spec and to validate your implementation.

Since there are now many extensions to the BitTorrent protocol, you should test with torrents that do not use new or experimental features. I have had good luck with torrents from [archive.org](https://archive.org/details/bittorrent) and [bt.etree.org](http://bt.etree.org/).

Good luck!

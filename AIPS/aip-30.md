---
  AIP: 30
  Title: P2P Communication Improvements - WebSockets
  Authors: Erwann Gentric <erwann@ark.io>
  Status: Draft
  Discussions-To: https://github.com/arkecosystem/AIPS/issues
  Type: Standards
  Category: Networking
  Created: 2019-04-03
  Last Update: 2019-04-04
--- 

## Abstract
This AIP describes the implementation of WebSockets as an improvement to P2P communication.

## Motivation
Current P2P communication is done through HTTP requests, with REST API like endpoints.

This isn't optimal for many reasons :
- No active connection to peers : every HTTP request is independant, which is good for most usages, but not suited for P2P communication
- Network load : every HTTP request comes with its header, adding up more data than necessary in P2P context
- Concurrent connections / requests : not optimal compared to WebSockets

WebSockets offer a solid alternative to HTTP in the P2P context.

## Rationale
For implementing the WebSocket layer, we decided to use SocketCluster : a framework for Node.JS with some nice features like auto-reconnection, middleware functions, scaling.

How we migrate from HTTP to WebSockets :
- We instanciate a SocketCluster server (instead of HTTP), and we configure endpoints to listen to (equivalent to former HTTP URLs)
- We pass data and headers in socket messages simply as an object { data, headers } (these are not HTTP headers, but some meta-information on the peer sending the socket message)
- Similar as with HTTP implementation, the endpoints are mapped to specific handlers : the socket message will be routed to corresponding handler which will proceed and return the eventual result
- Errors are handled through socket message error callback : no more HTTP status codes but "classical" error handling

## Specification

### Socket endpoints
The socket endpoints are defined as strings : it's up to us to decide on the format so that it makes sense.

We decided to go with a 3-part string :
- Prefix: always "p2p". It is good practice to have a specific prefix that will differentiate us from SockerCluster internal messages
- Main route: "peer" for peer messages, "internal" for forger-relay messages. Also used for internal logic are "utils" and "init" (only for worker-master communication, see more below)
- Method: specific method / handler to call. For example "getStatus"

So for example, in order to get a peer status, the endpoint to call would be "p2p.peer.getStatus".

### SocketCluster Master-Worker architecture

With SocketCluster, the "Master" process is responsible for starting up the socket application, initializing and managing sub-processes like the "Workers".

The workers are responsible for everything socket-related : connection, handling messages, middleware... They are launched as sub-processes from the master, and we can configure how much workers we want in order to best use our machine capabilities (number of cores).

Workers and Master communicate emitting messages. We use that to route peer requests from the workers to the master : indeed as of current Core implementation, the state of the application (blockchain, database, transaction pool...) is only accessible from the main process (through container), so the workers as independant sub-processes won't have access directly.

Here's a workflow to clarify :

Peer A => sends message to peer B : "p2p.peer.getStatus", with data { data, headers }

Peer B / worker => receives message, forwards to master { data, headers, endpoint }

Peer B / master => receives { data, headers, endpoint } from worker, proceeds and returns { data: { status }, headers }

Peer B / worker => sends back to Peer A { data: { status }, headers }

## Backwards Compatibility
Obviously, no backward compatibility is possible with former P2P implementation.

## Reference Implementation
Implementation is ready on core branch 2.4.

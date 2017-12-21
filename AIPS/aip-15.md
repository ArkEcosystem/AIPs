```
  AIP: 15
  Title: Event based subscriptions (WebHooks) for easier event chain listening on client side
  Authors: Kristjan Kosic - Chris <chris@ark.io>
  Status: Draft
  Type: Standards Track
  Created: 2017-12-21
  Last Update: 2017-12-21
```

Abstract
========

This AIP proposes the implementation of a WEBHOOK API interface for dApp developers to listen to ARK Blockchain in a simple manner (event subscription chain listeners).
In this way, the clients wouldn’t be polling the network for data, but would subscribe to an EVENT (see specifications below) via WEBHOOK system, thus taking some of the burden of the network.  When event happens the corresponding notifications will be sent to listeners.

Clients/users/dApp’s would define rules for EVENT in the config file/or api; 
ARK-NODE with listeners enabled would run in the background, waiting for events (event information data with accompanying payload) and 
Deliver them to all listeners via webhook notifications (e.g. callback in various forms).

Benefits
==========

Configured listeners would be running in the background. Web hooks are also especially important for IoT where smaller devices can listen and handle the events 

Users/developers - would get simple lightweight and efficient rule based event-subscription that notifies the subscribers of “events” without the need of installing additional tools, servers, etc... just to listen to the chain. Developers get event based notification; and they can continue to work with their dApp from there. 

For End Users - more dApps, more integration points, bigger adoption, easier integration with legacy systems.

Scenario:
==========
Precondition: ark-node is running and listening to ARK blockchain.
User specifies event conditions for the node to listen to in simple .json config file
ARK-NODE saves the event subscription and starts to listens to events. Conditions will be checked on every received block/tx.
When SUBSCRIBED_EVENT == TRUE; all the listeners are notified via the specified subscription conditions (callback URL [get, post] with accompanying JSON payload describing the SUBSCRIBED_EVENT data)
Event will stay stored, and status of acceptance is monitored and resent if not accepted by callback_url party.

Event Specifications:
===========
The listing below includes the basic events that the ARK Ecosystem users  can subscribe to.
Based on user specifications, the webhook notifications will be sent out, on every block update/tx/received or other timing events build upon ARK blockchain. By using this combinations users could easily set various rules (related to IOT development, just monitoring, alarm notifications, received tx monitoring, etc)...
More events would be added later on.

All Events would consist of:
- Conditions of event
- Callback URI
- Event Payload for the listeners (what to send with notification)

Some webhook examples below:
    Vendor Field alert WebHook 
    Conditions
    Vendor Field Value TBD (sub conditions)
    Sender
    Recipient
    Amount
    TimeStamp
    Callback URL
    Structure response

Transaction alert WebHook
    Conditions
    Amount
    Recipient
    Sender
    Vendor Field
    Type
    (Callback URL)
    Structure response (TBD)

Block Forged Alert WebHook
    Conditions
    Forger public Key
    Callback URL
    Structure response 
    Forged BlockID

Not Forging Alert WebHook
    Conditions:
        Forger public key
    Callback URL
    Structure response

Vote Alert WebHook
    Conditions
    PublicKey
    Address
    Callback URL
    Structure response


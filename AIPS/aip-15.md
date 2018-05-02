```
  AIP: 15
  Title: Event based subscriptions (WEBHOOKs) for easier event chain listening on client side
  Authors: Kristjan Kosic - Chris <chris@ark.io>
  Status: Draft
  Type: Standards Track
  Created: 2017-12-21
  Last Update: 2018-02-08
```

Abstract
========

This AIP proposes the implementation of a WEBHOOK API interface for dApp developers to listen to ARK Blockchain in a simple manner (event subscription chain listeners). Pooling is just wastefull and not efficient (neither of client or server side).

By delivering WEBHOOK functionality, the clients wouldn’t be polling the network for data, but would subscribe to an EVENT (see specifications below) via WEBHOOK API system, thus taking the burden of the network.  When event happens the corresponding notifications will be sent to listeners. Special nodes/or dedicated nodes can be started to deliver this functionality.

Clients/users/dApp’s would define rules for EVENT in the config file/or api; 
ARK-NODE with listeners enabled would run in the background, waiting for events (event information data with accompanying payload) and 
Deliver them to all listeners via webhook notifications (e.g. callback in various forms).

Benefits
==========

Configured listeners would be running in the background. WEBHOOK are also very important for IoT where smaller devices can listen and handle the events.

Users/developers - would get simple lightweight and efficient rule based event-subscription that notifies the subscribers of “events” without the need of installing additional tools, servers, etc... just to listen to the chain. Developers get event based notification; and they can continue to work with their dApp from there. 

For End Users - more dApps, more integration points, bigger adoption, easier integration with legacy systems/ecommmerce and other users.

Usage
==========
Precondition: ark-node is running and listening to ARK blockchain.
User specifies event conditions for the WEBHOOK and sends them to the specified ark-node via REST api call. If ark-node is configured to enable WEBHOOK and the specified security token is correct, subscription is saved in the database and response returned to the user, that the subscription is active.

For example (basic hook payload):
```
"hooks": [
        {
            "CallbackURI": "http://localhost/tx-alert",
            "Type" : "POST",
            "Payload" : "TBD-TX_Object",
            "HookType":"REST",
            "TriggerRule" : [
                { 
                    "RecipientID"  : "Axxaasd",
                    "Type"         : 2,
                    "SenderID"     : "asdasda"
                },
                "and" : {...},
                "or" : {...},
             ]
        }
    ]
```

ARK-NODE saves the event subscription and starts to check for events upon blocks. Conditions will be checked on every received block/tx.
When SUBSCRIBED_EVENT == TRUE; all the listeners are notified via the specified subscription conditions (callback URL [get, post] with accompanying JSON payload describing the SUBSCRIBED_EVENT data)
Event will stay stored, and status of acceptance is monitored and resent if not accepted by callback_url party.

Security
===========
- A whitelisting option will be added to settings, so only specific hooks/callbacks are allowed.
- A local listening node can be setup, alerting only the clients the node owner specifies
- A public test node will also be made available
- A secret token/hash will be defined by the ark-node. Users subscribing to webhooks, will need to set the token in headers (other subscription calls will be declined).
- Expiration time (hook subscription would need to be renewed every XX-time (defined in ark-node hook api properties))

Implementation
============
A special  configuration section in ark-node api. that defines webhook properties. If enabled a REST API listener would be set up - to listen to client-side hook subscriptions.

For example, a webhook config on the ark-node would look like this:
```
web-hooks: {
    enabled: true,
    options: {
      expiration-time: 72, //subscription must be renewed every XX hours
      expiration-active: true,
      secret-token: "80fff6600885bb75ac5f6a32cfdce60d",
      whitelist: ["127.0.0.1", "8.8.8.8"],
      alert-retry: 4, //number of alert retries
    }
  },
```

Upon succesfully received hook, the event conditions would be stored in the db and checked upon. A new table would be added to the database to store the hooks, and their callback statutes. Also to clean the hooks an expiration time is proposed. If the client doesn't confirm the hook in the specified time - it get's removed.

Event Specifications
===========
The listing below includes the basic events that the ARK Ecosystem users  can subscribe to.
Based on user specifications, the webhook notifications will be sent out, on every block update/tx/received or other timing events build upon ARK blockchain. By using this combinations users could easily set various rules (related to IoT development, just monitoring, alarm notifications, received tx monitoring, etc).

More events would be added later on.

All Events would consist of:
- Conditions of event
- Callback URI
- Event Payload for the listeners (what to send with notification)

Some webhook event examples below:

Vendor Field alert WebHook 
- Conditions
- Vendor Field Value TBD (sub conditions)
- Sender
- Recipient
- Amount
- TimeStamp
- Callback URL
- Structure response

Transaction alert WebHook
- Conditions
- Amount
- Recipient
- Sender
- Vendor Field
- Type
- (Callback URL)
- Structure response (TBD)

Block Forged Alert WebHook
- Conditions
- Forger public Key
- Callback URL
- Structure response 
- Forged BlockID

Not Forging Alert WebHook
- Conditions:
  -   Forger public key
- Callback URL
- Structure response

Vote Alert WebHook
- Conditions
- PublicKey
- Address
- Callback URL
- Structure response


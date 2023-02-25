---
categories: ["Configuration"]
tags: ["validator", "sentry", "config.toml","docs"]
title: "Validator Sentry config.toml"
linkTitle: "Validator-Sentry config.toml"
date: 2023-02-08
weight: 41
description: >
 How to setup your validator and sentry nodes
---


## Local Configuration Validator-Sentry Setup

The validator will only talk to the Sentry nodes that are provided. While sentry nodes have the ability to talk to the validator node on the private channel and talk to public nodes elsewhere on the internet. When initializing nodes there are six parameters in the config.toml which are important

* **pex**: boolean value. It tursn the peer exchange reactor (gossip protocol) on or off in a node. When **<code>pex=false</code>**, only the list of nodes in the <code>persistent_peers</code> list are available for connection.
* **persistent_peers**: commma separated list of [**nodeid@ip:port**](#nodeids) values that define a list of peers that expected to be online at all times and the node is expected to be able to connect to them. If some nodes are not available, they will be skipped and later retried for a while, before completely dropping them. If no nodes are available from the list and **<code>pex=false</code>**, then the node will not be able to join the network.
* **private_peer_ids**: comma-separated list of nodeid values, that should not be gossiped at all times. This setting tells which nodes should not be handed out to others
* **add_book_strict**: Turn this off if some of the nodes are on a LAN IP. If off, non routable IP address, like addressed on a private network can be added to the address book.
* **unconditional_peer_ids**: comma seperated list of nodeIDs. These nodes will be connected no matter the limit of inbound and outbound peers. This is useful when address_books of sentry nodes are full.
* **seed_mode**. This is used to jump start other nodes by providing a list of peers that the node can connect to.

### NodeIDs {#nodeids}
**Important**: Nodes in **<code>persistent_peers</code>** have the format nodeID@ip:port.  
 - **nodeID**. The ID of your validator. This can be retrieved with the command

    `ayad tendermint show-node-id --home /opt/aya`

    or use the nodebase showkeys script
    `./showkeys.sh`
- **ip**  The IP-address of your relay node
- **port**:  The p2p port your relay node is listening on. By default it is 26656
  
For the **<code>private_peer_ids</code>** and **<code>unconditional_peer_ids</code>**, only put the nodeID (without the @ip:port)

### Validator Node Configuration

{{<table table_class="table-striped table-hover" thead_class="table-dark">}}| Config Option | Setting |
|----|-----|
| seed_mode| false|
| pex | false |
|persistent_peers | list of sentry nodes |
| private_peer_ids | none |
| unconditional_peer_ids | optionally sentry node IDs |
|addr_book_strict | false |
{{</table>}}
>The validator node should have <code>pex=false</code> so it does not gossip to the entire network. The <code>persistent_peers</code> will be your sentry nodes. <code>private_peer_ids</code> can be left empty, as the validator is not trying to hide who it is communicating with. Setting <code>unconditional_peer_ids</code> is optional for a validator since they will not have a full address book. 
>Since the validator is on a private network and it will connect to the sentry nodes also on a private network, <code>addr_book_strict=false</code> has to be set.


### Sentry Node Configuration
{{<table table_class="table-striped table-hover" thead_class="table-dark">}}
| Config Option | Setting |
|----|-----|
| seed_mode| false |
| pex | true |
|persistent_peers | validator node, optionally other sentry nodes |
| private_peer_ids | validator node ID |
| unconditional_peer_ids | validator node ID, optionally other sentry node IDs |
|addr_book_strict | false |
{{</table>}}

>The sentry nodes should be able to talk to the entire network hence why **<code>pex=true</code>**.
 The **<code>persistent_peers</code>** of a sentry node will be the validator, and optionally other sentry nodes. 
 The sentry nodes should make sure they do not gossip the validator's ip,
  to do this you must put the validator nodeID as a **<code>private_peer_ids</code>**. The **<code>unconditional peer IDs</code>** will be the validator node ID and optionally other sentry node IDs.

**IMPORTANT** Do not forget to secure your node's firewalls when setting them up.

### About Gossipping
Nodes will gossip their known peers that  are not marked as a private node to other nodes on the p2p network.  
will be gossipped.

If you plan to install sentry nodes,  the recommendation is to setup the sentry first, then after that install the validator node and directly
connect it to the relay. If your validator node was already installed, it's already gossipped to other peers. And you want to prevent that.
You can fix this by blocking the p2p ports to non-relays with a firewall, and the peers will eventually remove your validator.
But ideally, the installation of a validator should be done with relay setup in mind to make it a complete private installation. This requires some handshaking.

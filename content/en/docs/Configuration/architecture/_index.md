---
categories: ["Configuration"]
tags: ["validator", "sentry", "docs"]
title: "Node Architecture Design"
linkTitle: "Node Architecture"
date: 2023-02-08
weight: 40
description: >
  An architectural overview on how to configure your validator and sentry nodes
resources:
- src:  "*.{png,jpg}"
  title: " Image #:counter"
  params:
   byline: "picture"
---

{{< toc >}}

# Validator Node Architecture Design

Tendermint relies on the set of validators to secure the network. The role of the validators is to run a full-node and participate in consensus by broadcasting votes which contain cryptographic signatures signed by their private key.  Validators are connected to each other through the **p2p** connections.

Validators are **responsible** for ensuring the that the network can sustain denial of service attacks.

## Common Attacks
- Distributed Denial of Service (DDoS)

    In the WM Chain, a validator node has a fixed IP address and RESRfule API port connected to the internet, which makes it vulnereable. DDoS attacks will halt the vote messages between validators and prevent blocks from being committed. If more than 1/3 of the network suffer from a DDoS attack it can halt the chain.

- Compromise of keys

    The most valuable asset of a validator are the keys it uses to sign blocks. An attacker who has control of the validator can get anything they want signed by the keys.

# Single Node Validator Setup

This approach is deemed unsafe, though you could set firewall white listings to establish links to only trustful peers. If an IP address is discovered it becomes vulnerable, and it is problematic to change. 

- **Pro**: easy to implement
- **Con**: not a flexible setup

> => This approach should be avoided, since it will still make your validator vulnerable.

{{< imgproc single-node Fit "600x500" >}}
{{< /imgproc >}}


# Single Layer Sentry Node Setup

In this setup, the validator node hides behind its two-layer sentry nodes. Only sentry nodes use public internet. The validator nodes do not need a public IP address, so discovery of IP addresses to target with DDos is much harder. Multiple Sentry Nodes can be connected to a a single validator to further mitigate the risk of DDoS Attacks.

# **Virtual Private Cloud (VPC)**
  
If you use a cloud solution, you can setup a Virtual Private Cloud (VPC) to host your relay and validator nodes. VPC networks provide a more secure connection between resources because  the inaccessible from the public internet and other VPC networks. Traffic within a VP network doesn't count against bandwidth usage.

   >In this setup, the relay talks to the validator >node through a secure private connection. The >relay node will then also connect through an >**external_address** with the p2p validator >network.

{{< imgproc single-layer Fit "600x500" >}}
{{< /imgproc >}}

  - **Pro**: Efficient to mitigate DDos attacks
  - **Con**: Attacker can only attack the validator node, if they gain access to the private network.


# Public Cloud 

In this setup, the relay and validator node are both connected to the internet. But the validator node is hidden behind the relay node. The relay node will prevent **gossipping** the IP-address of the validator node. 

>This is less secure than using a VPC network, >since the validator node, although unknown to the >rest of the peers, is still connected to the >public internet and could be discovered.


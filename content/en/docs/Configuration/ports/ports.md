---
categories: ["Configuration"]
tags: ["validator", "sentry", "docs"]
title: "Used Ports"
linkTitle: "Used Ports"
weight: 42
date: 2023-02-08
description: >
  What ports does **aya** use?
---

By default the follow ports are used by **aya**
* **26656**
  - pnp networking port to connect to the tendermint network
  - On a validator this port needs to be exposed to sentry nodes
  - On a sentry node this port needs to be exposed to the open internet
  - This is the port validators and sentries use to communicate to each other
- **26657** 
  - Tendermint RPC port.
  - Some tools, example querying tools, use this port to query blockchain status.
  - You should not directly expose this port. 
  - This should be shield from the open internet. 
- **26658**
  - Out of process ABCI app
  - This should be shielded from the open internet
- **29090**
  - the gRPC server port. This is used for gRPC server communication with
  - Cosmos SDK application layer. With this you can query banking, staking and - delegation information from Cosmos SDK.
  - Example, some monitoring services might use this communication mechanism
  - since its much faster than RESTful. 
  - This should be shielded from the open internet

Some optional ports that might be used by **aya** are as follows

  - **26600**
    - [Prometheus](https://github.com/tendermint/tendermint/blob/master/docs/nodes/metrics.md) stats server
    - System stats about the ayad process
    - Needs to be enabled in the config file
    - This should be shielded from the open internet
  - **1317**
    - The REST server 
    - for automated management of anything you can do with the CLI
    - This should be shieled from the open internet.

>More [here](https://docs.cosmos.network/main/core/grpc_rest) on  endpoints

>**Note**: Monitoring tools that are installed use additional port, ex Grafana uses port 3000


>**IMPORTANT**: 
For ports 1317 and 26657, you way want to use them as a RESTful endpoint by proxy-ing them to an external interface via an http proxy like Nginx or Coddy. The idea is to treat them like a web service for RESTful requests to the RPC ports done over https. You may configure rate limiting on the http proxy or make request with user authentication using a web app.  This way you are not exposing the ports directly and  your node is protected from receiving too many request suddenly.


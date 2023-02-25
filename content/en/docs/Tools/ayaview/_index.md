---
categories: ["Tools"]
tags: ["console", "ayaview", "docs"]
title: "Ayaview"
linkTitle: "Ayaview"
date: 2023-02-23
weight: 40
description: >
  Ayaview Console
resources:
- src:  "*.{png,jpg}"
  title: " Image #:counter"
  params:
   byline: "picture"  
---

## Introduction

Ayaview is a local monitoring tool for your validator node.It can be used besides a setup
based on Grafana-Prometheus for light-weight direct monitoring.

It has a very intuitive console view, written entirely in Golang for performance. 
Ayaview monitors your validator status and blocks produced on chain, as also system metrics and peer information. 

It can be run entirely standalone, started/stopped as many times you want without affecting your node.

## Overview

{{< imgproc ayaview Fit "900x600" >}}
{{< /imgproc >}}

## Installation Instructions

Download ayaview from nodebasewm github and unzip as follows

{{< highlight go "linenos=table,style=witchhazel" >}}
wget -O ayaview.zip https://github.com/nodebasewm/download/blob/main/ayaview.zip?raw=true
unzip ayaview.zip
{{< /highlight >}}

After configurating ayaview using the configuration instructions,
you can start ayaview in any open terminal window
{{< highlight go "linenos=table,style=witchhazel" >}}
./ayaview --config config.toml
{{< /highlight >}}


## Configuration

The configuration of ayaview is done through a **config.toml** configuration file. 
A default config file is delivered part of the installation


### Validator Address
> You will have to replace the value of **validator** with the address of your validator node.

To retrieve your validator's node address, take a look at the [Q&A in Testnet](/docs/testnet/qa/)

Replace the entire value of between the two "" with the address of your validator, ex.
{{< highlight go "linenos=table,style=witchhazel" >}}
validator="ayavaloper1r5v6c2ps2qqytdu7zcawwng7d5p5xm5g64cr07"
{{< /highlight >}}


### GRPC/RPC Ports
If you changed the default port of your node's GRPC (29090) or RPC endpoint (26657), you will have
to also change the grpc-address or rpc-address of the config.

Optionally, you can control the interval to update system metrics and node status. Block and consensus
updates are updated real-time.

{{< highlight go "linenos=table,style=witchhazel" >}}
# Bech prefixes for network.
bech-prefix = "aya"

#validator to watch FILL-IN 
validator="ayavaloperxx"

# If a network has specific bech prefixes for validator and for consensus node
# and their pubkeys, it's possible to specify them separately.
bech-validator-prefix = ""
bech-validator-pubkey-prefix = ""
bech-consensus-node-prefix = ""
bech-consensus-node-pubkey-prefix = ""

# system update interval, in seconds. Defaults to 3
interval = 3

# Node config.
[node]
# gRPC node address to get signing info and validators info from, defaults to localhost:9090
grpc-address = "localhost:29090"
# Tendermint RPC node to get block info from. Defaults to http://localhost:26657.
rpc-address = "localhost:26657"

# Logging config.
[log]
# Log level. Defaults to 'info', you can set it to 'debug' or even 'trace'
# to make it more verbose.
level = "info"
# Log all output in JSON except for fatal errors, useful if you are using
# logging aggregation solutions such as ELK stack.
json = true
{{< /highlight >}}


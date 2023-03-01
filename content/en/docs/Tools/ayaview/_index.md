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

{{< toc >}}

# Introduction

Ayaview is a local monitoring tool for your validator node.It can be used besides a setup
based on Grafana-Prometheus for light-weight direct monitoring.

It has a very intuitive console view, written entirely in Golang for performance. 
Ayaview monitors your validator status and blocks produced on chain, as also system metrics and peer information. 

It can be run entirely standalone, started/stopped as many times you want without affecting your node.

# Overview

{{< imgproc ayaview Fit "900x600" >}}
{{< /imgproc >}}

# Installation Instructions

Download ayaview from nodebasewm github and unzip as follows

{{< highlight go "linenos=table,style=witchhazel" >}}
wget -O ayaview.zip https://github.com/nodebasewm/download/blob/main/ayaview.zip?raw=true
unzip ayaview.zip
{{< /highlight >}}

After configurating ayaview using the configuration instructions,
you can start ayaview in any open terminal window.

{{< highlight go "linenos=table,style=witchhazel" >}}
./ayaview 
{{< /highlight >}}

A configuration file is provided, which must be used in the following conditions:
* You run ayaview on a sentry but want to see node status of your validator. 
* You changed the default ports of grpc (29090) or rpc (25567).
* You want the change the default update interval for system metrics

Then call ayaview as follows

{{< highlight go "linenos=table,style=witchhazel" >}}
./ayaview --config config.toml
{{< /highlight >}}

# Configuration

The configuration of ayaview is done through a **config.toml** configuration file. 
A default config file is delivered part of the installation.
> Note: This is **NOT** the same config.toml as the *aya config.toml*. Ayaview has its own config.toml file, which is part of the zip package.

## Ayaview on Sentry Node

If your run ayaview on a sentry node, you will only see chain activity such as blocks and chai n events. 

If you want to see the validator node status information this sentry is pointing to, 
you will have to pass a config.toml file to ayaview.

You can use the config.toml delivered as part of ayaview.zip.
Edit the value of **validator** in the config.toml with the address of your validator node.

This address is the **ayavalop** BECH32 notation of your validator node. 
It must start with the ayavalop prefix, any other BECH32 notation will **NOT** work.

To retrieve your validators operator address, take a look at the [Q&A in Testnet](/docs/testnet/qa/)

ex.

{{< highlight go "linenos=table,style=witchhazel" >}}
validator="ayavaloper1r5v6c2ps2qqytdu7zcawwng7d5p5xm5g64cr07"
{{< /highlight >}}

## Ayaview on Validator Node

When running ayaview on a validator node, it will now detect  that your node is a validator,
and configure itself automatically without the need of a config file.

So no configuration file is needed, unless off course you intent to observe a different node, 
or changed ports.

## GRPC/RPC Ports
If you changed the default port of your node's GRPC (29090) or RPC endpoint (26657), you will have to also change the grpc-address or rpc-address in the config file and pass the config file to ayaview.

## System update Interval

Optionally, you can control the interval to update system metrics and node status. Block and consensus
updates are updated real-time.

## Config-File
{{< highlight go "linenos=table,style=witchhazel" >}}
# Bech prefixes for network.
bech-prefix = "aya"

#validator to watch FILL-IN 
validator=""

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


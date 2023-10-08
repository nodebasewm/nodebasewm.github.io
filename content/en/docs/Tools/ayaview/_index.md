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

**Developed by Nodebase Team member [EarthNodeChile (Nico Verboven)](https://twitter.com/EarthNodeChile)**

{{< toc >}}

# Introduction

Ayaview is a local monitoring tool for your validator node.It can be used besides a setup
based on Grafana-Prometheus for light-weight direct monitoring.

It has a very intuitive console view, written entirely in Golang for performance. 
Ayaview monitors your validator status and blocks produced on chain, as also system metrics and peer information. 

It can be run entirely standalone, started/stopped as many times you want without affecting your node.

# AyaView

Ayaview provides node status display of your node. Uptime / peers / blocks proposed  and system metrics.

{{< imgproc ayaview Fit "900x600" >}}
{{< /imgproc >}}

# Validators

The validator view display active validators and their current status.
The information displayed is 
1. Moniker and address
2. MissedBlocks : How many blocks missed in slashing window
3. CCF : Checks if validator is on the Cardano ChainFollower list
4. Tokens: How many delegated tokens
5. Power: Voting power
6. State: Signing state
7. Commission: Commission rate set on this validator 


{{< imgproc validators Fit "900x600" >}}
{{< /imgproc >}}

Additionally, the view supports filtering and sorting.

{{< imgproc filtersort Fit "900x600" >}}
{{< /imgproc >}}

## Sorting Validator View

**Press the key 's'** to sort on different columns and order. By repeating
pressing the 's' key you will be able to cycle through different sort options,
display in the upper right corner

## Filter Validator View

Wildcard filtering is also added. The filter implements glob based style filtering (similar to file path filters on command line)

**By pressing the key 'f'** you enter the filter node, and you can start typing your filter,
display in the upper right corner, The section will be highhlighted in yellow.
And the filter is applied while typing in real-time updates. Because the filter captures the input, you need to press the ESC button to end filter setup, so you can go back and change the tab display

* Important: Only use the filtering patterns '*'  and '?'. The filter currently doesnt support [abc]  and [a-z] style filters

# Peers Display

A powerful feature of ayaview is to show peer connections by pressing the key 'p'. This will show you all peer connections with their latency and location. Also a live view of the P2P connection details are shown when selecting a peer, like  transaction speed/s.

{{< imgproc peers Fit "900x600" >}}
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
* You run ayaview on a sentry but want to see node status of your validator. (blocks signed/proposed ) 
* You changed the default ports of grpc (29090) or rpc (25567) or api (1317)
* You want the change the default update interval for system metrics
* You want to have a default filter applied to validators

Use ayaview with config file :

{{< highlight go "linenos=table,style=witchhazel" >}}
./ayaview --config config.toml
{{< /highlight >}}

If you run ayaview on your validator, it will automatically discover your validator node and connect to it.

# Configuration

The configuration of ayaview is done through a **config.toml** configuration file. A default config file is delivered part of the installation. 

{{% alert title ="Note" %}}

 This is **NOT** the same config.toml as the *aya config.toml*. Ayaview has its own config.toml file, which is part of the zip package.
{{% /alert %}}

## Ayaview on Sentry Node

If your run ayaview on a sentry node, you will only see chain activity such as blocks and chain events. If you want to see the validator node status information this sentry is pointing to, you will have to pass a config.toml file to ayaview. Ayaview will clearly display on the Home tab if your node is a validator or a sentry node.

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

So no configuration file is needed, unless off course you intent to observe a different node, or changed ports.

## GRPC/RPC/API Ports
If you changed the default port of your node's GRPC (29090) or RPC endpoint (26657), you will have to also change the grpc-address or rpc-address in the config file and pass the config file to ayaview.

The API part (default 1317) is needed to query WorldMobile specific APIs,
such as the chainfollower information. YOu will have to enable this API in
the ayad configuration app.toml under [api] section.


## System update Interval

Optionally, you can control the interval to update system metrics and node status. Block and consensus updates are updated real-time.

## Config-File
{{< highlight go "linenos=table,style=witchhazel" >}}
# Bech prefixes for network.
bech-prefix = "aya"

#validator to watch
validator=""

# If a network has specific bech prefixes for validator and for consensus node
# and their pubkeys, it's possible to specify them separately.
bech-validator-prefix = ""
bech-validator-pubkey-prefix = ""
bech-consensus-node-prefix = ""
bech-consensus-node-pubkey-prefix = ""

# system update interval, in seconds. Defaults to 3
interval = 3

# default filter for validators display Default is no-filtering
filter = ""

# Endpoint config.
[[Endpoint]]
name = "endpoint-2"
url = "localhost"
# gRPC node address to get signing info and validators info from, defaults to 29090
grpc_port = "29090"
# Tendermint RPC port  to get block info from. Defaults to 26657.
rpc_port = "26657"
# API config port for REST API, to enable edit app.toml of ayad 
# and go to [api] section and change enable to true
rest_port = "1317"

# Logging config.
[log]
# Log level. Defaults to 'info', you can set it to 'debug' or even 'trace'
# to make it more verbose.
level = "disabled"
# Log all output in JSON except for fatal errors, useful if you are using
# logging aggregation solutions such as ELK stack.
json = false

{{< /highlight >}}


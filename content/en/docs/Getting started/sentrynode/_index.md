---
categories: ["Getting Started"]
tags: ["validator", "sentry", "docs"]
title: "Sentry Node Installation"
linkTitle: "Sentry Node Installation"
date: 2023-02-13
weight: 40
description: >
  Installation Guide with script
---

{{< toc >}}

# Installation
Installing a sentry node requires a different installation procedure than 
a validor node.

The install_node.sh script provided by the WM team only covers validator nodes, and does not apply for sentry nodes. With some small adjustments
this script can be adjusted to be used to install sentry nodes

Sentry nodes dont have operator keys, since they are not part of the consensus voting.

## NodebaseTools
>The NodeBaseDevs team created an **install_sentry.sh** installation script.

This script is part of the **bundled nodebase tools package** release package that can be found in the [github releases](https://github.com/nodebasewm/download/releases/)

## Instructions
- Download the nodebase tools zipfile

  `wget https://github.com/nodebasewm/download/archive/refs/tags/v0.1.4-alpha.zip`

- unzip this file, it will create the folder download-0.1.3-alpha.
- copy the install_sentry.sh to your earthnode_installation folder
- The other scripts in the zip you can put wherever you want. Recommendation is to put them in /usr/local/bin
- make sure executable permissions are set on the scripts

    `chmod +x install_sentry.sh`
- run the script with a moniker name for your sentry from within the earthnode installation folder
  
  `./install_sentry.sh -m MONIKER_NAME` 

- Unlike a validator node, **no registration** is needed for sentry nodes.

- wait for the node to be synced to full height from the snapshot point.  **The installation script will terminate automatically when the node is synced to latest block height**

## Configuration
 
>Please continue with following the Sentry-Validator configuration instructions in [Configuration](/docs/configuration/config)  to configure your nodes accordingly after installation completes.

## Step-by-Step Example

>A step-by-step guide with example will be added to the Tutorials.

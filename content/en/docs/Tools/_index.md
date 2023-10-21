
---
title: "Tools"
linkTitle: "Tools"
weight: 3
date: 2017-01-05
description: >
  NodeBaseDevs Tools
---

The NodeBaseDevs team is continuously adding scripting and tooling to its [github release page](https://github.com/nodebasewm/download/releases/)

You can download the entire tool package as follows.
 {{< highlight go "linenos=table,style=witchhazel" >}}
wget https://github.com/nodebasewm/download/archive/refs/tags/nodebasetools.zip
 {{< /highlight >}}

The package currently contains:
- **install_sentry.sh** installation script
- **addresses.sh** : shows all your node address, useful for setup
- **validators.sh**: show all bonded validators by moniker/address and status
- **ayafix.sh**:   post installation fix to setup cosmovisor.service and fix links
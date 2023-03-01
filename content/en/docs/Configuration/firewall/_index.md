---
categories: ["Configuration"]
tags: ["validator", "sentry", "docs"]
title: "Firewall Settings"
linkTitle: "Firewall Settings"
weight: 43
date: 2023-02-08
description: >
  How to configure your firewall in a validator-sentry configuration
---

These instructions assume a Validator-Sentry architecture.

## Validator Node

Apply these firewall settings on your validator node


{{< highlight go "linenos=table,style=dracula" >}}
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow proto tcp from any to any port 22
sudo ufw allow from <sentry-ip> to any port 26656 proto tcp
{{< /highlight>}}


This allows ssh login, you can change **any** to an **IP-address** for higher security,

Only the sentry nodes can talk to the validator.
Repeat for each sentry node you have, the allow from sentry-ip to port 26656. If you have <code>prometheus=true</code> metrics logging enabled in the **config.toml**, you should allow the prometheus listen port as well.

When done

{{< highlight go "linenos=table,style=dracula" >}}
sudo ufw enable
sudo ufw reload
sudo systemctl restart ssh
sudo ufw status
{{< /highlight>}}

## Sentry Node

Apply these firewall settings on your sentry node

{{< highlight go "linenos=table,style=dracula" >}}
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow proto tcp from any to any port 22
sudo ufw allow from any to any port 26656 proto tcp
{{< /highlight>}}

Since the sentry node has pex enabled, connections to the p2p port (26656) need to be allowed, so that
other nodes can talk to the sentry.

When done

{{< highlight go "linenos=table,style=dracula" >}}
sudo ufw enable
sudo ufw reload
sudo systemctl restart ssh
sudo ufw status
{{< /highlight>}}

## Grafana
Addtional services can be opened such as grafana as follows. 
{{< highlight go "linenos=table,style=dracula" >}}
sudo ufw allow from any to any port 3000 proto tcp
{{< /highlight>}}

When done

{{< highlight go "linenos=table,style=dracula" >}}
sudo ufw enable
sudo ufw reload
sudo systemctl restart ssh
sudo ufw status
{{< /highlight>}}


## Prometheus
If you use Prometheus metrics, you should consult with the metrics exporter tool which port it uses for exporting
metrics. Prometheus specific setup recommendations and installation instructions  will be released shortly,
when the nodebase releases its nodebase monitoring service.
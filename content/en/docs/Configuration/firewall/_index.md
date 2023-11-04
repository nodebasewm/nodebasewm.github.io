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

## Sentry Node

Apply these firewall settings on your sentry node

{{< highlight go "linenos=table,style=dracula" >}}
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow proto tcp from <IP> to any port 22
sudo ufw allow from any to any port 26656 proto tcp
{{< /highlight>}}

Since the sentry node has **pex** enabled, connections from any machine to the p2p port (26656) needs to be allowed, 
so that other nodes can talk to the sentry for p2p communication of consensus messages.

The firewall settings above only allows ssh login from the given machine with IP-address  **IP**, 
which should be the client machine you want to login from. This is good practice and limits the attack vector on your server. 

>Note that with a proper VPN solution, such as tailscale, this specific firewall rule can be entirely omitted, 
giving you the highest level of security.

When done

{{< highlight go "linenos=table,style=dracula" >}}
sudo ufw enable
sudo ufw reload
sudo systemctl restart ssh
sudo ufw status
{{< /highlight>}}

## Validator Node

Apply these firewall settings on your validator node

{{< highlight go "linenos=table,style=dracula" >}}
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow proto tcp from <IP> to any port 22
{{< /highlight>}}

The validator firewall will be configured to make **only outgoing p2p connections** to the sentry nodes. 
Only allowing outgoing p2p connections to the sentry nodes further limits the attack vector on your validator node, 
since we don't have to open the p2p port. See the manual setup guide on how to configure the sentry/validator correctly for this. 

The firewall settings above only allows ssh login from the given machine with the give IP-address **IP**, 
which should be the client machine you want to login from.This is good practice and limits the attack vector on your server.  

> Note that with a proper VPN solution, such as tailscale, this specific firewall rule can be entirely omitted, giving you the highest level of security.

If you have <code>prometheus=true</code> metrics logging enabled in the **config.toml**, you should allow the prometheus listen port as well.

{{< highlight go "linenos=table,style=dracula" >}}
sudo ufw allow proto tcp from <IP> to any port 26600
{{< /highlight>}}

where **IP** is the address of your prometheus server.

When done

{{< highlight go "linenos=table,style=dracula" >}}
sudo ufw enable
sudo ufw reload
sudo systemctl restart ssh
sudo ufw status
{{< /highlight>}}


## Grafana

Additional services can be opened such as grafana as follows. I don't recommend to run your grafana server on the same machine as your validator. 
This should be discouraged. Grafana typically uses port 3000.

{{< highlight go "linenos=table,style=dracula" >}}
sudo ufw allow proto tcp from <IP> to any port 3000
{{< /highlight>}}

When done

{{< highlight go "linenos=table,style=dracula" >}}
sudo ufw enable
sudo ufw reload
sudo systemctl restart ssh
sudo ufw status
{{< /highlight>}}


## Prometheus

If you want to use Prometheus metrics, you should consult with the metrics exporter tool which port it uses for exporting
metrics. Prometheus specific setup recommendations and installation instructions  will be released shortly,
when the nodebase releases its nodebase monitoring service.
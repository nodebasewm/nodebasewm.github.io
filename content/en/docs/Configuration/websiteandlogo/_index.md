---
categories: ["Configuration"]
tags: ["validator", "website", "logo", "docs"]
title: "Update Website and Logo"
linkTitle: "Update Website and Logo"
weight: 43
date: 2023-04-04
description: >
  How to Update your ENO's Website link and Logo for them to appear online
---

**Written by Community Contributer [Cardano SHAMROCK](https://twitter.com/PoolShamrock)**

## Update Website And Description

1. On your Validator Node run

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ayad tx staking edit-validator --website "https://www.example.com"  --details "ENO Description" --from ACCOUNT --home /opt/aya
    {{< /highlight>}}

    > Note: Ensure to replace **ACCOUNT** with your own ENO's Account name, replace **https://www.example.com** with your own site address, replace **ENO Description** with your own preferred details.

## Update Logo

1. Create Account on Keybase: https://keybase.io

2. Add logo to Keybase

3. Copy Key from Keybase dashboard

4. On your Validator Node run

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ayad tx staking edit-validator --identity=KEYBASEKEY --from ACCOUNT --home /opt/aya
    {{< /highlight>}}

   > Note: Ensure to replace **ACCOUNT** with your own ENO's Account name, replace **KEYBASEKEY** with the key you obtained while signing up for Keybase

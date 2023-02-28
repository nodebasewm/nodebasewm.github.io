
---
toc: true
categories: ["Testnet"]
tags: ["setup", "install", "cosmovisor", "ayad"]
title: "Q&A"
linkTitle: "Q&A"
date: 2023-02-18
weight: 40
description: >
  Questions and Answers all things Testnet
---

{{< toc >}}

# I don't have a cosmovisor service? 
The cosmovisor service is a daemon process to keep your node running. When the ayad process would exit, the service would automatically restart. So it has a vital function to keep your validator running healthy.

The cosmovisor service is installed during the node installation (install_node.sh), after your registered your node with the ENNFT. If you failed to register the node, and stopped the installation script, the installation didnt complete.

You can still register the ENNFT following the published instructions, but the
cosmovisor service you would need to configure manually.

ayafix.sh is also part of the nodebase [tools](/docs/tools/)

Or you can install it manually from here as follows:

1. copy the content below to a file ex. ayafix.sh 
2. chmod +x ayafix.sh
3. sudo ./ayafix.sh
   
{{< highlight go "linenos=table,style=witchhazel" >}}
#!/usr/bin/env bash

aya_home=/opt/aya

echo -e "-- Configuring your node to start on server startup\n"
sudo ln -s $aya_home/cosmovisor/current/bin/ayad /usr/local/bin/ayad >/dev/null 2>&1
sudo ln -s $aya_home/cosmovisor/cosmovisor /usr/local/bin/cosmovisor >/dev/null 2>&1

sudo tee /etc/systemd/system/cosmovisor.service > /dev/null <<EOF
[Unit]
Description=Aya Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --home "$aya_home" &>>"$aya_home"/logs/aya.log
Restart=always
RestartSec=3
LimitNOFILE=4096

Environment="/opt/aya"
Environment="DAEMON_NAME=ayad"
Environment="DAEMON_DATA_BACKUP_DIR=/opt/aya/backup"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_HOME=/opt/aya"
[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable cosmovisor
{{< /highlight>}}

# What is my validator address?

The validator address is a function of the validator's public key, which is used to sign prevotes and precommits. 

Cosmos uses a bech32 encoded validator address, which is a more human-readable version of the same public key.

To retrieve your node's address you can use our script **addresses.sh** in [tools](/docs/tools/) or do it manually as follows.

{{< highlight go "linenos=table,style=witchhazel" >}}
ayad keys list --home /opt/aya
Enter keyring passphrase:
{{< /highlight>}}

The output will look like:
{{< highlight go "linenos=table,style=witchhazel" >}}

- address: aya1fnxullwf0r72hlmjyyuhjca23pl09a7rf5f752
  name: EarthNodeChile
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"A7tC1Yexx3ithv611VdPEySAffVP6nqTNvN9iKX1buMr"}'
  type: local
{{< /highlight >}}

With the address get the HEX presentation of it 

{{< highlight go "linenos=table,style=witchhazel" >}}
ayad keys parse aya1fnxullwf0r72hlmjyyuhjca23pl09a7rf5f752 --home /opt/aya
{{< /highlight >}}

{{< highlight go "linenos=table,style=witchhazel" >}}
bytes: 4CCDCFFDC978FCABFF7221397963AA887EF2F7C3
human: aya
{{< /highlight >}}

Now parse the HEX address to get all bech32 representations

{{< highlight go "linenos=table,style=witchhazel" >}}
ayad keys parse 4CCDCFFDC978FCABFF7221397963AA887EF2F7C3 --home /opt/aya
{{< /highlight >}}

The third entry with prefix **ayavaloper** is your validator address

{{< highlight go "linenos=table,style=witchhazel" >}}
formats:
- aya1fnxullwf0r72hlmjyyuhjca23pl09a7rf5f752
- ayapub1fnxullwf0r72hlmjyyuhjca23pl09a7rklpry4
- ayavaloper1fnxullwf0r72hlmjyyuhjca23pl09a7re7t8j5
- ayavaloperpub1fnxullwf0r72hlmjyyuhjca23pl09a7rng04ad
- ayavalcons1fnxullwf0r72hlmjyyuhjca23pl09a7rddcm74
- ayavalconspub1fnxullwf0r72hlmjyyuhjca23pl09a7rzv5ahu
{{< /highlight >}}


# How to add a profile image to your validator for WM Explorer?

**Written by Nodebase Team member [Gertjan](https://twitter.com/BKINDSPO)**

1. Go to the website https://keybase.io
2. Create an account. This can be done by going to "Login" and after that choosing for "Join Keybase"
3. Test if your account is working properly
4. Logout of your account. On the home page of Keybase click the button "Install"
5. Select the correct operating system and install the software
6. Start the Keybase client on you computer and login with the account you created in step 2
7. Change your profile image. This is the image that will show up in WM Explorer as your validator image
8. Create a PGP key by selecting "Add a PGP key"
9. Choose "Get a new PGP key"
10. Fill in the form and click "Let the math begin"
11. Click on the button "Done" in the dialog after
12. You can now see a 16 character PGP key
13. On your validator run the following command:

{{< highlight go "linenos=table,style=witchhazel" >}}
ayad tx staking edit-validator --identity="16 character PGP key" --from <operator name> --home /opt/aya/
{{< /highlight >}}

14. A transaction hash (txhash) is returned. Check in WM Explorer if the transaction was successful. If not, run
the command in step 13 again
15. After a successful transaction, your profile image will appear in WM Explorer
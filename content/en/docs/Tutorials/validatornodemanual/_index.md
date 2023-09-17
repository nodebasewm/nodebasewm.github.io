---
categories: ["Tutorial"]
tags: ["step-by-step", "validator", "docs"]
title: "2. Validator Node Manual Installation"
linkTitle: "2. Validator Node Manual Installation"
date: 2023-03-15
weight: 50
description: >
  Full Installation Guide for installing a Validator Node 
---

**Written by Nodebase Team member [intertree (Johnny Kelly)](https://twitter.com/intertreeJK)**

# Introduction

The following guide will take you through all manual steps to be taken to have a fully running Validator Node. Taking you through how attach it to your prevously installed Sentry Nodes, and how to register your EarthNode Stake Pool via both an On-Chain **ENNFT** Registration Transaction on the Cardano Network as well as an On-Chain **Validator** Registration Transaction on the Aya Network. 

A number of elements of the set up process are very similiar to that of setting up a Sentry Node. 

However additional steps will be required to ensure that the Validator only ever connects to other Nodes under *your own* control, so that your Validator Node's ID and IP adddress are never publicised to the rest of the Blockchain Network. 

This will require changing settings on **both** your existing Sentry Nodes and your newly set up Validator, to ensure proper cross connections are in place. 

Details of what changes need to be made, on both sides, will appear in this guide.

This guide assumes you have already set up the Sentry Nodes that will be used to connect to this Validator using the steps laid out over at [Sentry Node Manual Installation](/docs/tutorials/sentrynodemanual/). 

> Note: In order for full Validator set-up to work without **ever** connecting to a World Mobile supplied Node, you will need to have **two** working Sentry Nodes in your Infrastructure. 
>
> As they will be acting as the **two** required RPC Servers to help kickstart your Validator syncing to the tip of the Chain.

This guide also assumes that all IP addresses being used to established cross connections between your Sentry Nodes and your Validator will be **Private** IP addresses, and that each will be visable to one another.

> Note: In order for this to be the case **all** of your Nodes will need to appear under the same IP subnet. To achieve this within an Infrastructure in which you have Nodes running across either *multiple* Cloud Providers or a *hybrid* mix of Cloud and Baremetal Servers you will need to install and run a **VPN Service** that will make each of your Nodes appear to eachother as being on the same local network. 
>
> Setting up of such a VPN Service is outside the scope of this guide, but it is **highly** recommended that a VPN Service be used in any set up where Nodes do not *already* have Private IPs, on the same local subnet, that can be utilised for the task. 
>
> If all of your Nodes are running on the **same** Cloud Provider it is highly likely that they will *already* be running on the same Private IP subnet, and that that your Cloud Provider will make thse Private IPs available to you, but it must be highlighted that relying on a single Cloud Provider for **all** of your Node Infrastructure also creates a single point of failure for your EarthNode Stake Pool. 
>
> The level of acceptance you have for this single point of failure risk, is up to you.

And finally, this guide assumes you have already followed steps to set up a new user named wmt to run the Sentry Node software, and that you have taken proper steps to secure this username from being logged into from external machines. 

> **Note: DO NOT follow this guide while logged in as the root user on your machine. This is bad practice. Make a sudo group user named wmt FIRST, and configure secure login to it using SSH keys.**

Now, with this introduction, and its various warnings, out of the way, we shall proceed with the guide!

# Step-By-Step Installation Guide

1. Logged in as user wmt we first start by making the base directory structure for World Mobile's ayad and Cosmovisor binaries that are to be installed on our Node.

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo mkdir -p /opt/aya
    sudo chown "${USER}:${USER}" /opt/aya
    mkdir -p /opt/aya/cosmovisor/genesis/bin
    mkdir -p /opt/aya/backup
    mkdir -p /opt/aya/logs
    mkdir -p /opt/aya/config
    {{< /highlight>}}

2. Next we set some environment variables so that further installation steps can be simplified.

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    CHAIN_ID="aya_preview_501"
    aya_home=/opt/aya
    cosmovisor_logfile=${aya_home}/logs/cosmovisor.log
    en_registration_json=${aya_home}/registration.json
    accountfile=${aya_home}/account
    {{< /highlight>}}

3. Now we set up our Node's Account name (a friendly name for Our Operator Wallet Address that will be used to run our EarthNode Stake Pool) and Moniker (a friendly name for our Validator Node that will appear On-Chain for people to Delegate to). We will also save our Account name to a file for future reference. 

    We do this by entering the following group of commands
    
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    account='<account>'
    moniker='<moniker>'
    echo "$account" > "$accountfile"
    {{< /highlight>}}
    > Note: We replace ```<account>``` and ```<moniker>``` with our own chosen names at this point, removing the surrounding <> 
    >They **can** be the same name.


4. Now we install a required prerequisite package (jq) for the successful completion of installation steps.

    We do this by entering the following command
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo apt-get update
    sudo apt-get -q install jq -y
    {{< /highlight>}}

5. Now we create an installation directory for the EarthNode installation files, navigate to it, download the installer zip file, install the unzip command (if not already installed to our OS), and extract the earthnode_installer archive.

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    mkdir ~/earthnode_installer
    cd ~/earthnode_installer
    wget https://github.com/max-hontar/aya-preview-binaries/releases/download/v0.4.1/aya_preview_501_installer_2023_09_04.zip
    sudo apt-get -q install unzip -y
    unzip aya_preview_501_installer_2023_09_04.zip
    rm aya_preview_501_installer_2023_09_04.zip
    {{< /highlight>}}
 
6. Now we confirm that the included binaries for ayad and cosmovisor match their release_checksums values as provided by World Mobile. Check that the output of both commands match one another. 

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sha256sum ayad cosmovisor
    cat release_checksums 
    {{< /highlight>}}

7. Following this confirmation step, we copy the ayad and cosmovisor binary files to their home locations for use in future operations.

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cp ~/earthnode_installer/ayad "${aya_home}"/cosmovisor/genesis/bin/ayad
    cp ~/earthnode_installer/cosmovisor "${aya_home}"/cosmovisor/cosmovisor
    {{< /highlight>}}

8. Now we initialise ayad to create all of the required configuration and set up files needed for running the cosmovisor and ayad binaries. 

    We do this by entering the following command
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ./ayad init "${moniker}" --chain-id $CHAIN_ID --home ${aya_home}
    {{< /highlight>}}

    We have now populated the /opt/aya directory and its subdirectories with the necessary files to configure our Node. 

9. Next we copy across the genesis.json file used to kickstart the aya_preview_501 Blockchain Connection.

    We do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cp ~/earthnode_installer/genesis.json "${aya_home}"/config/genesis.json
    {{< /highlight>}}

10. Now we are going to create a new Operator Account, and display its Seed Phrase.  

    We do this by entering the following group of commands

    > Note: During this step you will be asked to set up a spending password for your Account, this password will be needed when sending your On-Chain Validator Registration Transaction in a later step. Enter in a secure password to be used for this.

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # Create a new operator account and store the JSON output in the 'operator_json' variable
    operator_json=$(./ayad keys add "${account}" --output json --home ${aya_home})

    # Extract the address from the 'operator_json' variable and store it in the 'operator_address' variable
    operator_address=$(echo "$operator_json" | jq '.address' | sed 's/\"//g')

    # Display the mnemonic and address of the operator account
    echo -e "\n-- [ONLY FOR YOUR EYES] Store this information safely, the mnemonic is the only way to recover your account. \n"
    echo "$operator_json" | jq -M
    {{< /highlight>}}

    > Note: This Operator Account is similar to a Cardano Wallet, and as such it will have along with it a Seed Phrase to allow for its future recovery. It is **vitally** important that we keep this Seed Phrase in a safe and secure location in case we ever need to recover our Account on a different machine in the future. The Operator Account is what controls our EarthNode Stake Pool's Registration and Pool settings, so if we lose access to this Account we will lose access to our EarthNode Stake Pool altogether. 

11. Next we are going to need the obtain our Validator Node's Node ID in order to complete some work over on our already configured Sentry Nodes in the next section of the guide.

    We do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ./ayad tendermint show-node-id --home $aya_home
    {{< /highlight>}}

    > Note: At this point we want to copy and paste the output of this command to a separate, temporary, text file on our machine for use in the **Sentry Node** section of this guide. Which is starting now. 

Now we need to prepare our **Sentry Nodes** for connection from our Validator. 

> Note: During the following section of the guide work will be being done on **both** of our Sentry Nodes to make them ready for Validator first contact. However, to simplify the instructions, the steps to take on our Sentry Nodes will only be run through once.
>
> We need to **Make sure** to complete the steps below on **both** of our Sentry Nodes. To do this, we will need to connect to each of them in separate terminal windows. Remember, these edits are **not** to be done on our Validator Node. 
>
> We also need to make sure to **not** close our existing Validator Node terminal window while working on the separate terminal windows of our already ocnfigured Sentry Nodes. As doing so will break future steps in Valdiator set up.

12. After connecting in a **separate** terminal window to our already configured **Sentry Node** we are now going to obtain its Node ID.

    We do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ayad tendermint show-node-id --home /opt/aya
    {{< /highlight>}}

    > Note: At this point we want to copy and paste the output of this command to a separate temporary text file on our machine for use later in this guide. 

13. Continuing on our already configured **Sentry Node** we are now going to make an edit to its  **config.toml** file to allow for remote RPC connections.

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cd /opt/aya/config
    nano config.toml 
    {{< /highlight>}}

    We are now in the nano text editor, looking at the config.toml file contents for our already configured **Sentry Node**. 

    > Note: It is possible to search for the fields we need to edit more quickly by copying them from the below steps, pressing ctrl+w inside of nano in our terminal window, right clicking on the window, pasting the names of the values we need to edit, and pressing enter to jump to them. 
    >
    > We can also remove text blocks from a document that we wish to replace with new text blocks by holding down shift, selecting the existing rows we wish to remove, and then pressing ctrl+k to remove them. 
    >
    > We can then select and copy the replacement text from the steps shown in this guide below, and right click and paste the new settings into the terminal window. 
    >
    > We can undo any mistakes we've made while working inside nano by pressing alt+u

    We now make the necessary changes to this file as follows

    * Add our Validator's Node ID, Private IP address, and Port number to receive connections from our Sentry Node to **persistent_peers**. Making sure **not** to remove the existing World Mobile peers.
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # Comma separated list of nodes to keep persistent connections to
    persistent_peers = "692f6bb765ed3170db4fb5f5dfd27c54503d52d3@peer1-501.worldmobilelabs.com:26656,d1da4b1ad17ea35cf8c1713959b430a95743afcd@peer2-501.worldmobilelabs.com:36656,e1f9c6bd4f701e8233729ceadbd1690bd782fced@10.106.0.2:26656,<Validator Node ID>@<Validator.Node.Private.IP>:26656"
    {{< /highlight>}}
    > Note: We replace ```<Validator Node ID>``` with our separately copied Validator Node ID from above and ```<Validator.Node.Private.IP>``` with our Validator Node's IP at this point, removing the surrounding <>

    * Add our Validator's Node ID to **unconditional_peer_ids**  
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # List of node IDs, to which a connection will be (re)established ignoring any existing limits
    unconditional_peer_ids = "<Validator Node ID>"
    {{< /highlight>}}
    > Note: We replace ```<Validator Node ID>``` with our separately copied Validator Node ID from above at this point, removing the surrounding <>

    * Add our Validator's Node ID to **private_peer_ids**  
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # Comma separated list of peer IDs to keep private (will not be gossiped to other peers)
    private_peer_ids = "<Validator Node ID>"
    {{< /highlight>}}
    > Note: We replace ```<Validator Node ID>``` with our separately copied Validator Node ID from above at this point, removing the surrounding <>

    * Set the **laddr = "tcp://127.0.0.1:26657"** field to be **laddr = "tcp://0.0.0.0:26657"** in the **RPC Server Configuration Options** section of the file. 
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # TCP or UNIX socket address for the RPC server to listen on
    laddr = "tcp://0.0.0.0:26657"
    {{< /highlight>}}

    At this point the editing work to our already configured **Sentry Node's** **config.toml** file is done, and it can now be saved with these changes. 

    To save the **config.toml** file within nano editor we press **ctrl+x** and then **press y**, followed by **enter**. This will save the file with the same filename as before.

14. Next we may need to make a change to the **app.toml** file contents for our already configured **Sentry Node**. 

    Luckily, the app.toml file we need to edit is in the same directory as config.toml. 

    So we do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    nano app.toml
    {{< /highlight>}}

    We now make the necessary change to this file as follows

    * Set snapshot interval to 100 instead of 0 to activate the snapshot manager
    > Note: It is quite possible that our already configured **Sentry Node's** snapshot-interval is already set to be 100. If this is the case, no further changes are neccessary. 
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # snapshot-interval specifies the block interval at which local state sync snapshots are
    # taken (0 to disable).
    snapshot-interval = 100
    {{< /highlight>}}
    
    At this point the editing work to our **app.toml** file is done, and it can now be saved with these changes. 

    To save the app.toml file within nano editor we press **ctrl+x** and then **press y**, followed by **enter**. This will save the file with the same filename as before.

15. Now we are going to add a new Firewall Rule on our already configured **Sentry Node**  to Allow traffic from our Validator Node's Private IP address to come in over Port 26657.

    What command we need to enter to do this will depend on how our Server is set up to handle Firewall Rules. 

    If our Server is using **ufw** to handle Firewall Rules (most likely for most installs) we need to enter the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo ufw allow from <Validator.Private.IP.address> to any port 26657 proto tcp
    {{< /highlight>}}

    If our Server is using iptables to handle Firewall Rules we need to enter the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo iptables -I INPUT -p tcp -s <Validator.Private.IP.address> --dport 26657 -j ACCEPT
    sudo service iptables save
    {{< /highlight>}}

    > Note: We replace ```<Validator.Node.Private.IP>``` with our Validator Node's IP at this point, removing the surrounding <>

16. Now we need to check that our changes didn't do anything to prevent our already configured **Sentry Node** from being able to start up its cosmovisor Service. 

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo systemctl restart cosmovisor.service
    sudo systemctl status cosmovisor.service
    {{< /highlight>}}

    If all of our changes have not caused any issues with our already configured **Sentry Node's** cosmovisor Service we should see something very similar to the following output.

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ● cosmovisor.service - Aya Node
         Loaded: loaded (/etc/systemd/system/cosmovisor.service; enabled; vendor preset: enabled)
         Active: active (running) since Wed 2023-03-15 16:27:02 UTC; 6s ago
       Main PID: 122758 (cosmovisor)
          Tasks: 15 (limit: 4660)
         Memory: 568.5M
            CPU: 9.121s
         CGroup: /system.slice/cosmovisor.service
                 ├─122758 /usr/local/bin/cosmovisor run start --home /opt/aya "&>>/opt/aya/logs/aya.log"
                 └─122764 /opt/aya/cosmovisor/genesis/bin/ayad start --home /opt/aya "&>>/opt/aya/logs/aya.log"

    Mar 15 16:27:02 wmt-sentry1 systemd[1]: Started Aya Node.
    Mar 15 16:27:02 wmt-sentry1 cosmovisor[122758]: 4:27PM INF running app args=["start","--home","/opt/aya","\u0026\u003e\u003e/opt/aya/logs/aya.log"] module=cosmovisor path=/opt/aya/cosmovisor/genesis/bin/ayad
    Mar 15 16:27:08 wmt-sentry1 cosmovisor[122764]: 4:27PM ERR Error dialing peer err="dial tcp <Validator.Node.Private.IP>:26656: connect: connection refused" module=p2p
    {{< /highlight>}}
    
    > Note: Even though we see a *connect: connection refused* error at this point this is only due to the fact that we haven't yet started up our Validator for the first time. Seeing this error simply means our configuration has been laoded into our already configured **Sentry Node** successfully. Because we entered our Validator Node's ID into the *unconditional_peers* section of the config.toml file, this should not pose an issue.

We have now concluded our required edits to our already configured **Sentry Node's** set up. We need to make sure that we have completed the above steps to **both** of our Sentry Nodes. 

We can now to return to our Valdiator Node to complete the rest of its set up. 

17. Before running our **Validator Node** for the first time there are some initial configuration changes that need to be made to allow for smooth operation and connection the to aya_preview_501 Blockchain. 

    So now we navigate to the aya config folder and open the config.toml file for our Validator Node to make these changes.

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cd /opt/aya/config
    nano config.toml 
    {{< /highlight>}}

    We are now in the nano text editor, looking at the config.toml file contents for our Validator Node. 

    We now make the necessary changes to this file as follows

    * Change the **statesync** option to be **enable = true** instead of **false**
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    #######################################################
    ###         State Sync Configuration Options        ###
    #######################################################
    [statesync]
    # State sync rapidly bootstraps a new node by discovering, fetching, and restoring a state machine
    # snapshot from peers instead of fetching and replaying historical blocks. Requires some peers in
    # the network to take and serve state machine snapshots. State sync is not attempted if the node
    # has any local state (LastBlockHeight > 0). The node will have a truncated block history,
    # starting from the height of the snapshot.
    enable = true 
    {{< /highlight>}}

    * Change the **pex** option to be **pex = false** instead of **true**
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # Set true to enable the peer-exchange reactor
    pex = false
    {{< /highlight>}}

    * Change the **addr_book_strict** option to be **addr_book_strict = false** instead of **true**
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # Set true for strict address routability rules
    # Set false for private or local networks
    addr_book_strict = false
    {{< /highlight>}}

    * Set the **log_level** to **"error"** instead of **"info"** 
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    log_level = "error"
    {{< /highlight>}}

    * Set our already configured Sentry Nodes, that will seed Blockchain Data to our new Sentry Node, as persistent_peers
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # Comma separated list of nodes to keep persistent connections to
    persistent_peers = "<Sentry Node 1 Node ID>@<Sentry.Node1.Private.IP>:26656,<Sentry Node2 Node ID>@<Sentry.Node2.Private.IP>:26656"
    {{< /highlight>}}
    > Note: We replace ```<Sentry Node 1 ID>``` and ```<Sentry Node 2 ID>``` with our separately copied Sentry Node IDs from above, as well as ```<Sentry.Node1.Private.IP>``` and ```<Sentry.Node2.Private.IP>``` with each of our already configured Sentry Node's IPs, at this point, removing the surrounding <>

    * Add our alreay configured Sentry Node IDs to **unconditional_peer_ids**  
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # List of node IDs, to which a connection will be (re)established ignoring any existing limits
    unconditional_peer_ids = "<Sentry Node 1 ID>,<Sentry Node 2 ID>"
    {{< /highlight>}}
    > Note: We replace ```<Sentry Node 1 ID>``` and ```<Sentry Node 2 ID>``` with our separately copied Sentry Node IDs from above at this point, removing the surrounding <>

    At this point the initial editing work to our **config.toml** file is done, and it can now be saved with these changes. 

    To save the **config.toml** file within nano editor we press **ctrl+x** and then **press y**, followed by **enter**. This will save the file with the same filename as before.

    Next we need to make some initial changes to the **app.toml** file contents for our Validator Node. 

    Luckily, the app.toml file we need to edit is in the same directory as config.toml. 

    So we do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    nano app.toml
    {{< /highlight>}}

    We now make the necessary changes to this file as follows

    * Replace GRPC port to not overlap with standard Prometheus port, replacing 9090 with 29090
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # Address defines the gRPC server address to bind to.
    address = "0.0.0.0:29090"
    {{< /highlight>}}
    * Make sure that the gas price units for our network are set to be 0uswmt
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # The minimum gas prices a validator is willing to accept for processing a
    # transaction. A transaction's fees must meet the minimum of any denomination
    # specified in this config (e.g. 0.25token1;0.0001token2).
    minimum-gas-prices = "0uswmt"
    {{< /highlight>}}

    At this point the initial editing work to our **app.toml** file is done, and it can now be saved with these changes. 

    To save the app.toml file within nano editor we press **ctrl+x** and then **press y**, followed by **enter**. This will save the file with the same filename as before.

18. Now we need to export some environment variables to get our system ready to run our Validator Node for the first time. 

    We do this by entering the following group of commands
    
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    export DAEMON_NAME=ayad
    export DAEMON_HOME="${aya_home}"
    export DAEMON_DATA_BACKUP_DIR="${aya_home}"/backup
    export DAEMON_RESTART_AFTER_UPGRADE=true
    export DAEMON_ALLOW_DOWNLOAD_BINARIES=true
    ulimit -Sn 4096
    {{< /highlight>}}

19. Before proceeding to start up our Valdiator Node for the first time we will need to install some monitoring software to see what it is doing once active. 

    So we are first going to install some prerequisites for the monitoring software, and then install the software itself.

    We do this by entering the following group of commands
    
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cd ~/
    mkdir nodebase-tools 
    cd nodebase-tools
    wget -O ayaview.zip https://github.com/nodebasewm/download/blob/main/ayaview.zip?raw=true
    unzip ayaview.zip
    rm ayaview.zip
    {{< /highlight>}}

20. We will also quickly add some new Firewall Rules to our Server to **only** allow Incoming connections from our already configured Sentry Node's to come into our Validator Node once it has started. 

    What command we need to enter to do this will depend on how our Server is set up to handle Firewall Rules. 

    If our Server is using **ufw** to handle Firewall Rules (most likely for most installs) we need to enter the following command to accept Incoming connections over Port 26656 (a Node's default Port)

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo ufw allow from <Sentry.Node1.Private.IP> to any port 26656 proto tcp
    sudo ufw allow from <Sentry.Node2.Private.IP> to any port 26656 proto tcp
    {{< /highlight>}}

    If our Server is using iptables to handle Firewall Rules we need to enter the following group of commands to accept Incoming connections over Port 26656 (a Node's default Port)

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo iptables -I INPUT -p tcp -s <Sentry.Node1.Private.IP> --dport 26656 -j ACCEPT
    sudo iptables -I INPUT -p tcp -s <Sentry.Node2.Private.IP> --dport 26656 -j ACCEPT
    sudo service iptables save
    {{< /highlight>}}

    If we haven't yet set up our Firewall Rules at all, we can follow the steps laid out over at [Firewall Configuration](/docs/configuration/firewall/) to do this.

21. Now we shall set up some variables that will be used in the next step to allow us to quickly manually start up our Validator, and let it begin its statesync process.

    We do this by entering the following group of commands

    > Note: We need to **make sure** we have replaced  **<Sentry.Node1.Private.IP>** and **<Sentry.Node1.Private.IP>** below with the *Private IP addresses* that come from each of our Sentry Nodes, removing the surrounding **<>*s*, before entering this group of commands.
    >
    > We can do this by copying the above group of commands below into a separate, temporary, text file and making any required edits before finally copying and pasting them into our Validator Node's terminal window.  

    
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    RPC_PEER_1="<Sentry.Node1.Private.IP>"
    RPC_PEER_1="<Sentry.Node1.Private.IP>"
    {{< /highlight>}}

22. With the Firewall Rules, and our Sentry Nodes' Private IP addresses set, added we are now going to start up our Valdiator Node for the first time, manually. 

    > Note: Later we will be setting up a service file to have our Node automatically restart on Server reboot, or following a crash. For now though we will proceed manually.

    We do this by entering the following group of commands
    
    > Note: There is a lot of automated scripting shown below that has to be run before first boot of Node to ensure that it will start to sync the chain. It will not be needed again for future start ups.
    >
    > We need to **make sure** that this whole set of commands is copied and pasted into our Terminal window as a single, whole, blob of commands, and is not entered piecemeal, for the blob to work properly.

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    INTERVAL=100
    LATEST_HEIGHT=$(curl -s "${RPC_PEER_1}:26657/block" | jq -r .result.block.header.height)
    BLOCK_HEIGHT=$((($((LATEST_HEIGHT / INTERVAL)) - 1) * INTERVAL + $((INTERVAL / 2))))
    TRUST_HASH=$(curl -s "${RPC_PEER_1}:26657/block?height=${BLOCK_HEIGHT}" | jq -r .result.block_id.hash)

    # Set available RPC servers (at least two) required for light client snapshot verification
    sed -i -E "s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"${RPC_PEER_1}:26657,${RPC_PEER_2}:26657\"|" "${aya_home}"/config/config.toml
    # Set "safe" trusted block height
    sed -i -E "s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT|" "${aya_home}"/config/config.toml
    # Set "qsafe" trusted block hash
    sed -i -E "s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" "${aya_home}"/config/config.toml
    # Set trust period, should be ~2/3 unbonding time (3 weeks for preview network)
    sed -i -E "s|^(trust_period[[:space:]]+=[[:space:]]+).*$|\1\"302h0m0s\"|" "${aya_home}"/config/config.toml

    cd ~/earthnode_installer
    "${aya_home}"/cosmovisor/cosmovisor run start --home ${aya_home} &>>"${cosmovisor_logfile}" &
    {{< /highlight>}}

    We have now started our Node software for the first time!

23. Now we shall proceed to monitoring it.

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cd ~/nodebase-tools 
    ./ayaview
    {{< /highlight>}}

    The ayaview software will start up and visually show us the Blockchain data being downloaded to bring us up to date with the current tip of the chain. 

    We will know it has caught up to the current state of the Chain once the Block Height column stops counting up so quickly, and starts adding just 1 new Block Height to left hand table only around every 5-6 seconds.

    Once we are up to date with the current tip of the Chain we need to do some tidy up work on both the config.toml and app.toml files that we edited before the first run of our Node. 

    We can do this by first pressing q on the ayaview console to quit out of it and then by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cd /opt/aya/config/
    nano config.toml
    {{< /highlight>}}

    We are now back in the nano text editor, looking at the config.toml file contents for our Validator Node. 

    We now make the necessary changes to this file as follows

    * change the **statesync** option to be **enable = false** instead of **true**

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    #######################################################
    ###         State Sync Configuration Options        ###
    #######################################################
    [statesync]
    # State sync rapidly bootstraps a new node by discovering, fetching, and restoring a state machine
    # snapshot from peers instead of fetching and replaying historical blocks. Requires some peers in
    # the network to take and serve state machine snapshots. State sync is not attempted if the node
    # has any local state (LastBlockHeight > 0). The node will have a truncated block history,
    # starting from the height of the snapshot.
    enable = false
    {{< /highlight>}}

    At this point the tidy up editing work to our config.toml file is done, and it can now be saved with these changes. 

    To save the config.toml file within nano editor we press ctrl+x and then press y, followed by enter. This will save the file with the same filename as before.

    >Note: This **config.toml** file edit above should only be completed once we are SURE that our Node is up to date with the current state of the Chain. We DO NOT make this edit before our Node has fully synced to the current Block Height ayaview, and is adding a new Block around every 5-6 seconds. 

24. At present, our Node should be running along nicely in the background and keeping up to date with the current state of the Chain, but we still have to complete some more steps before we have fully completed our Valdiator Node's initial set up. 

    First we to ensure that we have saved all of our Node's important data, needed for future reference both by tools and by us.

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cd ~/earthnode_installer
    # Get the address of the validator
    validator_address=$(./ayad tendermint show-address --home ${aya_home})

    # Use 'jq' to create a JSON object with the 'moniker', 'operator_address' and 'validator_address' fields
    jq --arg key0 'moniker' \
       --arg value0 "$moniker" \
       --arg key1 'operator_address' \
       --arg value1 "$operator_address" \
       --arg key2 'validator_address' \
       --arg value2 "$validator_address" \
       '. | .[$key0]=$value0 | .[$key1]=$value1 | .[$key2]=$value2' \
     <<<'{}' | tee $en_registration_json    
     {{< /highlight>}}

    This will save our Validator Node's data to the filename registration.json in the **/opt/aya** directory.
  
25. Next we want to set up some symbolic links for the 'ayad' and 'cosmovisor' binaries so that their specific commands can be called from anywhere on our Node's filesystem.

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo ln -s $aya_home/cosmovisor/current/bin/ayad /usr/local/bin/ayad >/dev/null 2>&1
    sudo ln -s $aya_home/cosmovisor/cosmovisor /usr/local/bin/cosmovisor >/dev/null 2>&1
    {{< /highlight>}}

26. And finally, we want to create a systemd service file that will allow our Node to automatically start on a reboot of our Server and to automatically attempt to restart itself on any crashes.

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo tee /etc/systemd/system/cosmovisor.service > /dev/null <<EOF
    # Start the 'cosmovisor' daemon and append any output to the 'aya.log' file
    # Create a Systemd service file for the 'cosmovisor' daemon
    [Unit]
    Description=Aya Node
    After=network-online.target

    [Service]
    User=$USER
    # Start the 'cosmovisor' daemon with the 'run start' command and write output to 'aya.log' file
    ExecStart=$(which cosmovisor) run start --home "${aya_home}" &>>"${aya_home}/logs/aya.log"
    # Restart the service if it fails
    Restart=always
    # Restart the service after 3 seconds if it fails
    RestartSec=3
    # Set the maximum number of file descriptors
    LimitNOFILE=4096

    # Set environment variables for data backups, automatic downloading of binaries, and automatic restarts after upgrades
    Environment="DAEMON_NAME=ayad"
    Environment="DAEMON_HOME=${aya_home}"
    Environment="DAEMON_DATA_BACKUP_DIR=${aya_home}/backup"
    Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
    Environment="DAEMON_RESTART_AFTER_UPGRADE=true"

    [Install]
    # Start the service on system boot
    WantedBy=multi-user.target
    EOF
    {{< /highlight>}}

    This will have added a new service file to our Server under the path /etc/systemd/system named cosmovisor.service. This service file is what will be called when the Server reboots, or if our Node crashes.

27. With the service now created, we now need to enable it for future use.

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # Reload the Systemd daemon
    sudo systemctl daemon-reload
    # Enable the 'cosmovisor' service to start on system boot
    sudo systemctl enable cosmovisor
    {{< /highlight>}}

    We can now confirm that our service is ready to be started by entering the following command. 

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo systemctl status cosmovisor.service
    {{< /highlight>}}

    This should show us the following

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ○ cosmovisor.service - Aya Node
        Loaded: loaded (/etc/systemd/system/cosmovisor.service; enabled; vendor preset: enabled)
        Active: inactive (dead)
    {{< /highlight>}}

    If we see this, we know we have successfully installed our Aya Node Service. 

28. All that remains now, is to close down our first run of our Node and to restart it using this newly installed Service instead of manually launching our Node as before.

    To do this, we need to identify what the current process number of our still running first run Node is. 

    We do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ps -ef | grep cosmovisor
    {{< /highlight>}}

    The output of this command should something like this

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    wmt       147797  146916  0 16:44 pts/0    00:00:00 /opt/aya/cosmovisor/cosmovisor run start --home /opt/aya
    wmt       147803  147797 11 16:44 pts/0    00:01:05 /opt/aya/cosmovisor/genesis/bin/ayad start --home /opt/aya
    wmt       148235  146916  0 16:54 pts/0    00:00:00 grep --color=auto cosmovisor
    {{< /highlight>}}

    What we are looking for is the leftmost number appearing for the line that contains "/opt/aya/cosmovisor/cosmovisor run start --home /opt/aya" on its right hand side

    In the above example, this number is 147797. 

    Our number will be different, and we need to take a note of it for the next step. 

    Once we have this number we can now use it to turn off our first run Node that has served us so well to get our Blockchain sync up to date with the current tip of the Chain. 

    We do this by entering the following command
    Note: We replace ```<number>```   with our noted number from above at this point, removing the surrounding <>

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo kill <number>
    {{< /highlight>}}

    We have now killed our running Node, but don't worry, we are going to bring it right back to life. Except this time, as a Service! 

    To do this we enter the following command
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo systemctl start cosmovisor.service
    {{< /highlight>}}

    Now we need to check that cosmovisor.service has started properly.

    We do this by entering the following command 
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo systemctl status cosmovisor.service
    {{< /highlight>}}
    
    When we ran this command previously it said our Aya Node Service was inactive (dead), this time it should say something like the following 
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cosmovisor.service - Aya Node
        Loaded: loaded (/etc/systemd/system/cosmovisor.service; enabled; vendor preset: enabled)
        Active: active (running) since Wed 2023-03-01 04:06:05 UTC; 4s ago
    Main PID: 34791 (cosmovisor)
        Tasks: 15 (limit: 4573)
        Memory: 382.3M
            CPU: 5.063s
        CGroup: /system.slice/cosmovisor.service
                ├─34791 /usr/local/bin/cosmovisor run start --home /opt/aya "&>>/opt/aya/logs/aya.log"
                └─34796 /opt/aya/cosmovisor/genesis/bin/ayad start --home /opt/aya "&>>/opt/aya/logs/aya.log"

    Mar 01 04:06:05 localhost systemd[1]: Started Aya Node.
    Mar 01 04:06:05 localhost cosmovisor[34791]: 4:06AM INF running app args=["start","--home","/opt/aya","\u0026\u003e\u003e/opt/aya/logs/aya.log"] module=cosmovisor path=/opt/aya/cosmovisor/genesis/bin/ayad
    {{< /highlight>}}

    Some details will be different to the above example, but this should be the general layout. The important point is that it should say 'active (running)' in green.

29. We can now confirm everything is continuing to sync from the Blockchain by going back to our ayaview console and watching to see if the Block Height is still slowly ticking upwards as before.

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cd ~/nodebase-tools
    ./ayaview
    {{< /highlight>}}

    If everything is working as it should our ayaview console should now be showing us the Aya Blockchain ticking on by once more, around every 5-6 seconds.
    
At this stage in the guide, we have successfully completed setting up a Validator Node that is able to **privately** sync with the Aya Blockchain, through **only** our already configured Sentry Nodes, but we have **not** yet used it to Register our EarthNode Stake Pool to both the Cardano and Aya Network Blockchains. 

So, we shall now proceed to doing this. 

> Note: Before proceeding, we press q to leave ayaview. 

30. First we need to obtain the details needed for our ENNFT Registration on the Cardano Blockchain side of the equation. We have actually already been shown them earlier, but let's just get them again.

    We do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cat /opt/aya/registration.json
    {{< /highlight>}}

    The output of this command should look something like this

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    {
      "moniker": "<our Validator Node Moniker>",
      "operator_address": "<our Operator Address beginning aya1, followed by a random string of letters and numbers>",
      "validator_address": "<our Validator Address beginning ayavalcons1, followed by a random string of letters and numbers>"
    }
    {{< /highlight>}}

    > Note: At this point we want to copy and paste the output of this command, including its surrounding {}s, to a separate, temporary, text file on our machine. 

31. Now we need to open a web browser and navigate to https://tinyurl.com/wmeno and click on the **EarthNode Registration** tab at the top of the page.

32. Next we need to connect our Cardano Preview Testnet Wallet that contains our Testnet EarthNode NFT to be Registered to this EarthNode Registration site.

    We do this by 

    a) Ensuring the Wallet containing our Testnet EarthNode NFT is the one currently selected for dApp connections in our Cardano Wallet Browser Extension
    b) Esnuring that it has enough collateral set to enable sumbission of Smart Contract Transactions (5 ADA should be enough)
    and then
    c) By clicking on the Connect Wallet Button on the site EarthNode Registration site 

    > Note: Once our Preview Testnet Wallet is connected the **EarthNode NFTs in your Wallet:** section of the EarthNode Registration site should populate with the EarthNodeNFT# numbers that you hold in your Preview Testnet Wallet.

33. Now we need to click on the EarthNodeNFT# number we wish to associate with our newly set up Validator Node.

34. Next we make sure we have RegisterEarthNode selected under the **Select Transaction Type:** Dropdown Menu. 

35. And now we need to paste in the previously copied information, obtained during step 32 above, into the **Paste Installation Script Output:** Box and press the **Build Transaction** Button.

    > Note: We will now be asked for our Preview Testnet Wallet's Spending Password at this point in order to Authorise the Registration Transaction to be Posted On-Chain on the Cardano Preview Testnet Newtork.

36. We need to enter our Spending Password and click Sign. 

    This should then show us a **Transaction Successful!** message on the EarthNode Registration Page along with a Transaction Hash that corresponds to the Transaction ID that will appear on the Cardano Blockchain. 

    > Note: Whilst the Transaction itself has now been successfully submitted, it may still take a little bit of time for it to appear On the Cardano Blockchain. There will be a link at the bottom of the EarthNode Registration site that will take us to an external page to reload until we see the Transcation finally go through. Once we see this Transaction appear on that page. We can proceed to the next step.

37. Now we return to our Validator Node's Terminal Window as before, and complete the final steps to get our EarthNode Stake Pool online. The first thing we will do is make sure that our Operator Address received 5 uswmt Tokens to be able to complete the Aya Network side of our Stake Pool Registration. 

    We do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ayad query bank balances <our Operator Address> --home $aya_home
    {{< /highlight>}}

    > Note: We replace ```<our Operator Address>``` with the full operator_address detail that begins with *aya1* from the information we obtained for EarthNode Registration in step 32, removing the surrounding <>

    If Registration on the Cardano Blockchain side has been successful, this command should give us the following output

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    balances:
    - amount: "5"
      denom: uswmt
    pagination:
      next_key: null
      total: "0"
    {{< /highlight>}}

    > Note: A balance amount of "5" appearing in this output means that The Cardano Transaction triggered our Operator Address being topped up with 5 uswmt to be used to set up our EarthNode Stake Pool on the Aya Blockchain side. The side we are now working with. 

38. Now we need to Register our EarthNode Stake Pool to the Aya Blockchain Network. 

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ayad tx staking create-validator \
      --amount=1uswmt \
      --pubkey="$(ayad tendermint show-validator --home ${aya_home})" \
      --moniker="$moniker" \
      --chain-id="$CHAIN_ID" \
      --commission-rate="0.10" \
      --commission-max-rate="0.20" \
      --commission-max-change-rate="0.01" \
      --min-self-delegation="1" \
      --from="$account" \
      --home ${aya_home} \
      --output json \
      --yes 
    {{< /highlight>}}

    > Note: After entering this group of commands we will be asked to enter the Spending Password we set up earlier in the guide specifically for our Validator Account. We must enter it now.

    If the Transcation has been successful we should see an output that contains a txhash value that consists of a string of capital letters and numbers, along with a lot of other information in JSON format, that looks something like this

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    {"height":"0","txhash":"<Transaction Hash containing capital letters and numbers>","codespace":"","code":0,"data":"","raw_log":"[]","logs":[],"info":"","gas_wanted":"0","gas_used":"0","tx":null,"timestamp":"","events":[]}
    {{< /highlight>}}

29. Now we now need to go and see if our Validator has been successfully Registered on the Aya Blockchain Network by opening our browser and going to https://wmt-explorer.com/Testnet

    Once there we need to look at the 'New joined EarthNode Validators' section and check to see if the Monkier we gave our EarthNode Stake Pool has appeared in the list. 


40. Finally, we need to go back to ayaview and make sure the status of our Node has been updated to say that the Validator is now BONDED and Active On-Chain. Which should be visable in the upper left box of the ayaview console. 

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cd ~/nodebase-tools
    ./ayaview
    {{< /highlight>}}



If the Moniker appears, and ayaview reports what we are expecting to see, **Congratulations!** We have now successfully completed setting up both our Validator Node **AND** our EarthNode Stake Pool!

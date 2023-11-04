---
categories: ["Tutorial"]
tags: ["step-by-step", "sentry", "docs"]
title: "1. Sentry Node Manual Installation"
linkTitle: "1. Sentry Node Manual Installation"
date: 2023-03-02
weight: 40
description: >
  Full Installation Guide for installing a sentry node 
---

**Written by Nodebase Team member [intertree (Johnny Kelly)](https://twitter.com/intertreeJK)**

# Introduction

The following guide will take you through all manual steps to be taken to have a fully running Sentry Node ready to be attached to your later installed Validator.

This guide assumes you have already followed steps to set up a new user named wmt to run the Sentry Node software, and that you have taken proper steps to secure this username from being logged into from external machines. 

> **Note: DO NOT follow this guide while logged in as the root user on your machine. This is bad practice. Make a sudo group user named wmt FIRST, and configure secure login to it using SSH keys.**

Now, with that warning out of the way, we shall proceed with the guide!

# Step-By-Step Installation Guide

1. Logged in as user wmt we first start by making the base directory structure for World Mobile's ayad and Cosmovisor binaries that are to be installed on our Node.

    We do this by entering the following group of commands

    > Note: There is **no need** to replace ```"${USER}:${USER}"``` below with your own username, as ```${USER}``` is a **system variable** that will **always be** the currently logged on user. Which, in this case, **should be** user ```wmt```.
    
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo mkdir -p /opt/aya
    sudo chown "${USER}:${USER}" /opt/aya
    mkdir -p /opt/aya/cosmovisor/genesis/bin
    mkdir -p /opt/aya/backup
    mkdir -p /opt/aya/logs
    mkdir -p /opt/aya/config
    {{< /highlight>}} 

3. Next we set up our Node's Aya Network Chain ID *(depending on which Chain ID this Node will be running on)*

    We do this by entering the **one of** the following commands

    > Note: For the moment there is only **one** Chain ID for World Mobile Aya Chain Networks available publicly, but this will be changing soon and, as a result, there will be more than the single Chain ID option below to choose from. For now though, the single Chain ID option available is listed below.

    *For Aya Network Public Testnet:* 

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    CHAIN_ID="aya_preview_501"
    {{< /highlight>}}

4. Now we set up our Node's Moniker (a friendly name for our Node to help identify it) 

    We do this by entering the following command    
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    moniker='node'
    {{< /highlight>}}
    > Note: ```'node'``` is an **optional** default Moniker we can use to create some *'security by obscurity'* on the Aya Blockchain Network, by **NOT** having our **publicly listed** Sentry Node be named **in any way** that can be linked back to the name of our ENO. If a custom public Moniker is preferred, however, We can replace ```'node'``` with our own chosen name at this point, keeping the``` ''```s in place.


5. Now we install a required prerequisite package (jq) for the successful completion of installation steps.

    We do this by entering the following command
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo apt update
    sudo apt-get -q install jq -y
    {{< /highlight>}}

6. Now we create an installation directory for the EarthNode installation files, navigate to it, download the installer zip file, install the unzip command (if not already installed to our OS), and extract the earthnode_installer archive.

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    mkdir ~/earthnode_installer
    cd ~/earthnode_installer
    wget https://github.com/max-hontar/aya-preview-binaries/releases/download/v0.4.1/aya_preview_501_installer_2023_09_04.zip
    sudo apt-get -q install unzip -y
    unzip aya_preview_501_installer_2023_09_04.zip
    rm aya_preview_501_installer_2023_09_04.zip
    {{< /highlight>}}
 
7. Now we confirm that the included binaries for ayad and cosmovisor match their release_checksums values as provided by World Mobile. Check that the output of both commands match one another. 

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sha256sum ayad cosmovisor
    cat release_checksums 
    {{< /highlight>}}

8. Following this confirmation step, we copy the ayad and cosmovisor binary files to their home locations for use in future operations.

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cp ~/earthnode_installer/ayad /opt/aya/cosmovisor/genesis/bin/ayad
    cp ~/earthnode_installer/cosmovisor /opt/aya/cosmovisor/cosmovisor
    {{< /highlight>}}

9. Now we initialise ayad to create all of the required configuration and set up files needed for running the cosmovisor and ayad binaries. 

    We do this by entering the following command
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ./ayad init "${moniker}" --chain-id $CHAIN_ID --home /opt/aya
    {{< /highlight>}}

    We have now populated the /opt/aya directory and its subdirectories with the necessary files to configure our Node. 

10. Next we copy across the genesis.json file used to kickstart the aya_preview_501 Blockchain Connection.

    We do this by entering the following command
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cp ~/earthnode_installer/genesis.json /opt/aya/config/genesis.json
    {{< /highlight>}}

11. Before running our Sentry Node for the first time there are some initial configuration changes that need to be made to allow for smooth operation and connection the to aya_preview_501 Blockchain, and to ensure that connections between our own ENO Infrastructure's Nodes remain robust. 

    So now we navigate to the aya config folder and open the config.toml file for our Sentry Node to make these changes.

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cd /opt/aya/config
    nano config.toml 
    {{< /highlight>}}

    We are now in the nano text editor, looking at the config.toml file contents for our Sentry Node. 

    >Note: It is possible to search for the fields we need to edit more quickly by copying them from the below steps, pressing ctrl+w inside of nano in our terminal window, right clicking on the window, pasting the names of the values we need to edit, and pressing enter to jump to them. 
    >
    > We can also remove text blocks from a document that we wish to replace with new text blocks by holding down shift, selecting the existing rows we wish to remove, and then pressing ctrl+k to remove them. 
    >
    >We can then select and copy the replacement text from the steps shown in this guide below, and right click and paste the new settings into the terminal window. 
    >
    >We can undo any mistakes we've made while working inside nano by pressing alt+u

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

    * Set the public nodes supplied by World Mobile, that provide a statesync of historical Blockchain Data to our new Sentry Node, as persistent_peers
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # Comma separated list of nodes to keep persistent connections to
    persistent_peers = "692f6bb765ed3170db4fb5f5dfd27c54503d52d3@peer1-501.worldmobilelabs.com:26656,d1da4b1ad17ea35cf8c1713959b430a95743afcd@peer2-501.worldmobilelabs.com:26656"
    {{< /highlight>}}

    * Set the public seed node supplied by World Mobile, that helps keep your Sentry Node's P2P address book file up to date, as a seed in **seeds**
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # Comma separated list of seed nodes to connect to
    seeds = "7836955a4d42ed85a6adb13ae4f96806ab2fd9b2@peer3-501.worldmobilelabs.com:26656"    
    {{< /highlight>}}


    * Set the **external_address = ""** field to have our own Node's Public IP address followed by our Node's local connection Port (default 26656), so that it can broadcast this information to other nodes over P2P (replace x.x.x.x with our Node's Public IP). 
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # Address to advertise to peers for them to dial
    # If empty, will use the same port as the laddr,
    # and will introspect on the listener or use UPnP
    # to figure out the address. ip and port are required
    # example: 159.89.10.97:26656
    external_address = "x.x.x.x:26656"
    {{< /highlight>}}

    At this point the initial editing work to our **config.toml** file is done, and it can now be saved with these changes. 

    To save the **config.toml** file within nano editor we press **ctrl+x** and then **press y**, followed by **enter**. This will save the file with the same filename as before.

    Next we need to make some initial changes to the **app.toml** file contents for our Sentry Node. 

    Luckily, the app.toml file we need to edit is in the same directory as config.toml. 

    So we do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    nano app.toml
    {{< /highlight>}}

    We now make the necessary changes to this file as follows

    * Replace GRPC port to not overlap with standard Prometheus port, replacing 9090 with 29090
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # Address defines the gRPC server address to bind to.
    address = "localhost:29090"
    {{< /highlight>}}
    * make sure the gas price units for our network to be 0uswmt
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # The minimum gas prices a validator is willing to accept for processing a
    # transaction. A transaction's fees must meet the minimum of any denomination
    # specified in this config (e.g. 0.25token1;0.0001token2).
    minimum-gas-prices = "0uswmt"
    {{< /highlight>}}
    * Change the API Configuration section **enable** option to be **enable = true** instead of **false**
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    [api]

    # Enable defines if the API server should be enabled.
    enable = true
    {{< /highlight>}}

    At this point the initial editing work to our **app.toml** file is done, and it can now be saved with these changes. 

    To save the app.toml file within nano editor we press **ctrl+x** and then **press y**, followed by **enter**. This will save the file with the same filename as before.

12. Now we need to export some environment variables to get our system ready to run our Sentry Node for the first time. 

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    export DAEMON_NAME=ayad
    export DAEMON_HOME=/opt/aya
    export DAEMON_DATA_BACKUP_DIR=/opt/aya/backup
    export DAEMON_RESTART_AFTER_UPGRADE=true
    export DAEMON_ALLOW_DOWNLOAD_BINARIES=true
    ulimit -Sn 4096
    {{< /highlight>}}

13. Before proceeding to start up our Sentry Node for the first time we will need to install some live monitoring software to see what it is doing once active. 

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cd ~/
    mkdir nodebase-tools 
    cd nodebase-tools
    wget -O ayaview.zip https://github.com/nodebasewm/download/blob/main/ayaview.zip?raw=true
    unzip ayaview.zip
    rm ayaview.zip
    {{< /highlight>}}

14. We will also quickly add a new Firewall Rule to our Server to allow Incoming connections to come into our Node once it has started. 

    What command we need to enter to do this will depend on how our Server is set up to handle Firewall Rules. 

    If our Server is using **ufw** to handle Firewall Rules (most likely for most installs) we need to enter the following command to accept Incoming connections over Port 26656 (a Sentry Node's default Port)

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo ufw allow from any to any port 26656 proto tcp
    {{< /highlight>}}

    If our Server is using iptables to handle Firewall Rules we need to enter the following group of commands to accept Incoming connections over Port 26656 (a Sentry Node's default Port)

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo iptables -I INPUT -p tcp -m tcp --dport 26656 -j ACCEPT
    sudo service iptables save
    {{< /highlight>}}

    If we haven't yet set up our Firewall Rules at all, we can follow the steps laid out over at [Firewall Configuration](/docs/configuration/firewall/)  to do this.

15. With the Firewall Rule added we are now going to start up our Sentry Node for the first time, manually. 

    >Note: Later we will be setting up a service file to have our Node automatically restart on Server reboot, or following a crash. For now though we will proceed manually.

    We do this by entering the following group of commands
    >Note: There is a lot of automated scripting shown below that has to be run before first boot of Node to ensure that it will start to sync the chain. It will not be needed again for future start ups.

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    INTERVAL=15000
    LATEST_HEIGHT=$(curl -s "http://peer1-501.worldmobilelabs.com:26657/block" | jq -r .result.block.header.height)
    BLOCK_HEIGHT=$((($((LATEST_HEIGHT / INTERVAL)) - 1) * INTERVAL + $((INTERVAL / 2))))
    TRUST_HASH=$(curl -s "http://peer1-501.worldmobilelabs.com:26657/block?height=${BLOCK_HEIGHT}" | jq -r .result.block_id.hash)

    # Set available RPC servers (at least two) required for light client snapshot verification
    sed -i -E "s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"http://peer1-501.worldmobilelabs.com:26657,http://peer2-501.worldmobilelabs.com:26657\"|" /opt/aya/config/config.toml
    # Set "safe" trusted block height
    sed -i -E "s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT|" /opt/aya/config/config.toml
    # Set "qsafe" trusted block hash
    sed -i -E "s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" /opt/aya/config/config.toml
    # Set trust period, should be ~2/3 unbonding time (3 weeks for preview network)
    sed -i -E "s|^(trust_period[[:space:]]+=[[:space:]]+).*$|\1\"302h0m0s\"|" /opt/aya/config/config.toml

    cd ~/earthnode_installer
    /opt/aya/cosmovisor/cosmovisor run start --home /opt/aya &>>/opt/aya/logs/cosmovisor.log &
    {{< /highlight>}}

    We have now started our Node software for the first time!

16. Now we shall proceed to monitoring it.

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

    We are now back in the nano text editor, looking at the config.toml file contents for our Sentry Node. 

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

17. Next we need to make a change to the app.toml file contents for our Sentry Node. 

    We do this by entering the following command
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    nano app.toml
    {{< /highlight>}}

    And we edit the following section of the file to the below setting, changing it from being 0 to being 100. 

    >Note: Setting the snapshot-interval to 100 will ensure that we can use our Sentry Node to kickstart our Validator later on. Which will allow us to set up our Validator without ever exposing its external IP to the rest of the Network.

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # snapshot-interval specifies the block interval at which local state sync snapshots are
    # taken (0 to disable).
    snapshot-interval = 100
    {{< /highlight>}}

    At this point the tidy up editing work to our app.toml file is done, and it can now be saved with these changes. 

    To save the app.toml file within nano editor we press ctrl+x and then press y, followed by enter. This will save the file with the same filename as before.

    >Note: These **config.toml** and **app.toml** file edits above should only be completed once we are SURE that our Node is up to date with the current state of the Chain. We DO NOT make this edit before our Node has fully synced to the current Block Height ayaview, and is adding a new Block around every 5-6 seconds. 

18. At present, our Node should be running along nicely in the background and keeping up to date with the current state of the Chain, but we still have to complete some more steps before we have fully completed our Sentry Node's initial set up. 

    First we need to ensure that we have saved all of our Node's important data, needed for future reference both by tools and by us.

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cd ~/earthnode_installer
    # Get the address of the validator
    validator_address=$(./ayad tendermint show-address --home /opt/aya)
    # Use 'jq' to create a JSON object with the 'moniker', 'operator_address' and 'validator_address' fields
    jq --arg key0 'moniker' \
    --arg value0 "$moniker" \
    --arg key1 'validator_address' \
    --arg value1 "$validator_address" \
    '. | .[$key0]=$value0 | .[$key1]=$value1'  \
    <<<'{}' | tee /opt/aya/sentry.json
    {{< /highlight>}}

    This will save our Sentry Node's data to the filename sentry.json in the **/opt/aya** directory.
  
19. Next we want to set up some symbolic links for the 'ayad' and 'cosmovisor' binaries so that their specific commands can be called from anywhere on our Node's filesystem.

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo ln -s /opt/aya/cosmovisor/current/bin/ayad /usr/local/bin/ayad >/dev/null 2>&1
    sudo ln -s /opt/aya/cosmovisor/cosmovisor /usr/local/bin/cosmovisor >/dev/null 2>&1
    {{< /highlight>}}

20. And finally, we want to create a systemd service file that will allow our Node to automatically start on a reboot of our Server and to automatically attempt to restart itself on any crashes.

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo tee /etc/systemd/system/cosmovisor.service > /dev/null <<EOF
    # Start the 'cosmovisor' daemon
    # Create a Systemd service file for the 'cosmovisor' daemon
    [Unit]
    Description=Aya Node
    After=network-online.target

    [Service]
    User=$USER
    # Start the 'cosmovisor' daemon with the 'run start' command and write output to journalctl
    ExecStart=$(which cosmovisor) run start --home /opt/aya
    # Restart the service if it fails
    Restart=always
    # Restart the service after 3 seconds if it fails
    RestartSec=3
    # Set the maximum number of file descriptors
    LimitNOFILE=4096

    # Set environment variables for data backups, automatic downloading of binaries, and automatic restarts after upgrades
    Environment="DAEMON_NAME=ayad"
    Environment="DAEMON_HOME=/opt/aya"
    Environment="DAEMON_DATA_BACKUP_DIR=/opt/aya/backup"
    Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
    Environment="DAEMON_RESTART_AFTER_UPGRADE=true"

    [Install]
    # Start the service on system boot
    WantedBy=multi-user.target
    EOF
    {{< /highlight>}}

    This will have added a new service file to our Server under the path /etc/systemd/system named cosmovisor.service. This service file is what will be called when the Server reboots, or if our Node crashes.

21. With the service now created, we now need to enable it for future use.

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

22. All that remains now, is to close down our first run of our Node and to restart it using this newly installed Service instead of manually launching our Node as before.

    To do this, we need to identify what the current process number of our still running first run Node is. 

    We do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ps -ef | grep cosmovisor
    {{< /highlight>}}

    The output of this command should something like this

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    wmt   33619   33486  0 03:11 pts/0    00:00:02 /opt/aya/cosmovisor/cosmovisor run start --home /opt/aya
    wmt   33625   33619 33 03:11 pts/0    00:16:16 /opt/aya/cosmovisor/genesis/bin/ayad start --home /opt/aya
    wmt   34703   33486  0 03:59 pts/0    00:00:00 grep --color=auto cosmovisor
    {{< /highlight>}}

    What we are looking for is the leftmost number appearing for the line that contains "/opt/aya/cosmovisor/cosmovisor run start --home /opt/aya" on its right hand side

    In the above example, this number is 33619. 

    Our number will be different, and we need to take a note of it for the next step. 

    Once we have this number we can now use it to turn off our first run Node that has served us so well to get our Blockchain sync up to date with the current tip of the Chain. 

    We do this by entering the following command
    >Note: We replace ```<number>```   with our noted number from above at this point, removing the surrounding <>

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
                ├─34791 /usr/local/bin/cosmovisor run start --home /opt/aya
                └─34796 /opt/aya/cosmovisor/genesis/bin/ayad start --home /opt/aya

    Mar 01 04:06:05 localhost systemd[1]: Started Aya Node.
    Mar 01 04:06:05 localhost cosmovisor[34791]: 4:06AM INF running app args=["start","--home","/opt/aya"] module=cosmovisor path=/opt/aya/cosmovisor/genesis/bin/ayad
    {{< /highlight>}}

    Some details will be different to the above example, but this should be the general layout. The important point is that it should say 'active (running)' in green.

23. We can now confirm everything is continuing to sync from the Blockchain by going back to our ayaview console and watching to see if the Block Height is still slowly ticking upwards as before.

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cd ~/nodebase-tools
    ./ayaview
    {{< /highlight>}}

    If everything is working as it should our ayaview console should now be showing us the Aya Blockchain ticking on by once more, around every 5-6 seconds.


**And that's it!!!**

**Congratulations! We have now successfully completed setting up a Sentry Node!**

> Note: At the moment though, our Sentry Node does not yet have a Validator that it connects to for protecting it, and passing on its Block Votes. 
>
>Which means it hasn't yet got to fulfil its true purpose in life just yet (much like many of us).
>
>So, to fix this, we will tackle connecting our newly set up Sentry Node to our EarthNode Validator, as well as any other running Sentry Nodes in our Infrastructure, in a separate guide. 
>
>For now though,  we can simply bask in the glory of setting up a well running Sentry Node! 

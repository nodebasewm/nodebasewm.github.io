---
categories: ["Tutorial"]
tags: ["step-by-step", "validator", "cardano", " "chain-follower", "docs"]
title: "3. Cardano Chain Follower Manual Installation and root-list Registration"
linkTitle: "3. Cardano Chain Follower Manual Installation and root-list Registration"
date: 2023-09-17
weight: 60
description: >
  Full Installation Guide for installing a Cardano Chain Follower Service and registering it to the Cardano Chain Follower root-list. 
---

**Written by Nodebase Team member [intertree (Johnny Kelly)](https://twitter.com/intertreeJK)**

# Introduction

The following guide will take you through all manual steps to be taken to have a fully running Cardano Chain Follower. Taking you through how attach it to an existing public Cardano API Server, as well as how to register your EarthNode to the Cardano Chain Follower root-list via 2 On-Chain Registration Transactions on the Aya Network. 

At the moment, this installation **only** requires set up through an existing On-Chain Registered **Validator** Node. In the future proxy connection methods may be introduced, but, for now, connections are directly made from your EarthNode Validator to a pre-configured Cardano API Server Endpoint. 

This guide assumes you have already set up both of your Sentry Nodes and your EarthNode Validator, and have Registered your Validator to the Chain, using the steps laid out over at [Sentry Node Manual Installation](/docs/tutorials/sentrynodemanual/) and [Validator Node Manual Installation](/docs/tutorials/validatornodemanual/), respectively. 

Now, with this introduction out of the way, we shall proceed with the guide!

# Step-By-Step Installation Guide

1. Logged in as user wmt we first start by making the base directory for the Cardano Chain Follower files that are to be installed on our Node.

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo mkdir -p /opt/aya/ccf
    sudo chown "${USER}:${USER}" /opt/aya/ccf
    {{< /highlight>}}

2. Next we set the aya_home environment variable so that further installation steps can be simplified.

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    aya_home=/opt/aya
    {{< /highlight>}}

3. Now we install a required prerequisite package (libpq-dev) for the successful completion of installation steps.

    We do this by entering the following group of commands
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo apt-get update
    sudo apt install libpq-dev -y
    {{< /highlight>}}

4. Now we create an installation directory for the Cardano Chain Follower installation files, navigate to it, download the installer zip file, install the unzip command (if not already installed to our OS), and extract the Cardano Chain Follower archive.

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    mkdir ~/earthnode_installer/
    mkdir ~/earthnode_installer/ccf
    cd ~/earthnode_installer/ccf
    wget https://cdn.discordapp.com/attachments/1072502970027081749/1144181313583186002/wm_230824_ccf_rev2.zip
    sudo apt-get -q install unzip -y
    unzip wm_230824_ccf_rev2.zip 
    rm wm_230824_ccf_rev2.zip
    {{< /highlight>}}
 
5. Now we copy the aya_chain_follower binary and daemon_wm.toml configuration files to their home locations for use in future operations.

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cp ~/earthnode_installer/ccf/aya_chain_follower "${aya_home}"/ccf
    cp ~/earthnode_installer/ccf/daemon_wm.toml "${aya_home}"/ccf
    {{< /highlight>}}


6. Next we want to create a systemd service file that will allow our Cardano Chain Follower to automatically start on a reboot of our Server and to automatically attempt to restart itself on any crashes.

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo tee /etc/systemd/system/chain_follower.service > /dev/null <<EOF
    # Create systemd service file that describes the background service running the 'aya_chain_follower' daemon.
    [Unit]
    Description=Cardano Chain Follower
    After=network-online.target

    [Service]
    # Execute daemon from user account
    User=$USER
    # Set working directory
    WorkingDirectory=${aya_home}/ccf/
    # Start the 'aya_chain_follower' daemon with providing configuration file path
    ExecStart=${aya_home}/ccf/aya_chain_follower daemon --config ${aya_home}/ccf/daemon_wm.toml
    # Restart the service if it fails
    Restart=always
    # Restart the service after 3 seconds if it fails
    RestartSec=3
 
    [Install]
    # Start the service on system boot
    WantedBy=multi-user.target
    EOF
    {{< /highlight>}}

    This will have added a new service file to our Server under the path /etc/systemd/system named chain_follower.service. This service file is what will be called when the Server reboots, or if our Service crashes.

7.  With the service now created, we now need to enable it for future use.

    We do this by entering the following group of commands

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    # Reload the Systemd daemon
    sudo systemctl daemon-reload
    # Enable the 'chain_follower' service to start on system boot
    sudo systemctl enable chain_follower
    {{< /highlight>}}

8.  We can now confirm that our service is ready to be started.

    We do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo systemctl status chain_follower
    {{< /highlight>}}

    This should show us the following

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ○ chain_follower.service - Cardano Chain Follower
         Loaded: loaded (/etc/systemd/system/chain_follower.service; enabled; vendor preset: enabled)
         Active: inactive (dead) 
    {{< /highlight>}}

    If we see this, we know we have successfully installed our Cardano Chain Follower Service. 

9.  Now we start the Cardano Chain Follower Service.

    We do this by entering the following command
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo systemctl start chain_follower.service
    {{< /highlight>}}

10. Now we need to check that chain_follower.service has started properly.

    We do this by entering the following command 

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    sudo systemctl status chain_follower.service
    {{< /highlight>}}
    
    When we ran this command previously it said our Cardano Chain Follower Service was inactive (dead), this time it should say something like the following 
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ● chain_follower.service - Cardano Chain Follower
         Loaded: loaded (/etc/systemd/system/chain_follower.service; enabled; vendor preset: enabled)
         Active: active (running) since Fri 2023-09-08 14:52:46 UTC; 3s ago
       Main PID: 105986 (aya_chain_follo)
          Tasks: 15 (limit: 38243)
         Memory: 4.7M
            CPU: 96ms
         CGroup: /system.slice/chain_follower.service
                 └─105986 /opt/aya/ccf/aya_chain_follower daemon --config /opt/aya/ccf/daemon_wm.toml

    Sep 08 14:52:46 eno318-validator aya_chain_follower[105986]: /api/info/asset/metadata/{fingerprint}
    Sep 08 14:52:46 eno318-validator aya_chain_follower[105986]: /api/info/history/discover/{hash}
    Sep 08 14:52:46 eno318-validator aya_chain_follower[105986]: /api/info/history/address/
    Sep 08 14:52:46 eno318-validator aya_chain_follower[105986]: /api/info/address/stake/assets/
    Sep 08 14:52:46 eno318-validator aya_chain_follower[105986]: /api/info/addresses/assets/
    Sep 08 14:52:46 eno318-validator aya_chain_follower[105986]: /api/info/pools/{page}
    Sep 08 14:52:46 eno318-validator aya_chain_follower[105986]: /api/info/tokens/isNft/
    Sep 08 14:52:46 eno318-validator aya_chain_follower[105986]: /api/aya/epoch/change/from/{epoch1}/{epoch2}
    Sep 08 14:52:46 eno318-validator aya_chain_follower[105986]: /api/aya/epoch/change/latest
    Sep 08 14:52:46 eno318-validator aya_chain_follower[105986]: /api/aya/epoch/current/
    {{< /highlight>}}

    Some details will be different to the above example, but this should be the general layout. The important point is that it should say 'active (running)' in green and there should be a list of '/api/' items.

At this stage in the guide, we have successfully completed setting up a Cardano Chain Follower Service that is able to **publically** call the IOG Provided Cardano Testnet API Server *(a potential privacy concern which *may* be addressed later by World Mobile in their guidance on how to set up these Services)*, but we have **not** yet used it to Register our EarthNode to the Cardano Chain Follower root-list. 

So, we shall now proceed to doing this. 

> Note: Before proceeding, we press ctrl+c to leave the Service status view. 

10. First we need to obtain our account name that was previously saved during out Validator's set up process.

    We do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    cat /opt/aya/account
    {{< /highlight>}}

    > Note: The output of this command will be the name to replace the word ```<account>``` with in the next step.

11. Now we need submit our 1st of 2 Transactions to complete formal Registration to be a part of the Cardano Chain Follower **Root list**.

    > Note: The Root list is a list of **all** viable Registered Cardano Chain Follwer Validators that can be selected from to become a part of the **Leader set** in any given Epoch. 
    >
    > The Leader set are the smaller, rotating, sub-group of Validators from the overall root-list of Registered Validators that have the job of monitoring for specific Transaction information on the Cardano Blockchain, interpreting that information, and carrying out any actions that need to be taken on the Aya Network Blockchain as a result of that information.
    >
    > **Which** Registered Cardano Chain Folllower Validators get to become a part of the *current* Leader set is determined by the Aya Chain itself, on a rotational basis.

    We do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ayad tx cce set-val-acc --home /opt/aya --from <account> --chain-id aya_preview_501 -y
    {{< /highlight>}}

    > Note: After entering this command we will be asked to enter the Spending Password we set up earlierin the previous guide used specifically to set up our Validator Account. We must enter it now.
    >
    > Also, remember to replace <account> with the output of the previously completed cat command used to get this information. Also ensure to remove the **<>s** from the placeholder text above.

    If the Transcation has been successful we should see an output that contains a txhash value that consists of a string of capital letters and numbers, along with a lot of other information, that looks something like this

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    code: 0
    codespace: ""
    data: ""
    events: []
    gas_used: "0"
    gas_wanted: "0"
    height: "0"
    info: ""
    logs: []
    raw_log: '[]'
    timestamp: ""
    tx: null
    txhash: <Transaction Hash containing capital letters and numbers>
    {{< /highlight>}}

12. Before proceeding to the next step, we need to wait long enough so that the 1st Registration step fully completed as expected. We are waiting for the Transaction that was just posted to the Aya Blockchain to make its way to a fully Minted Block, and to form a part of that Block's list of Transactions. 

    We can look to see if and when this Transaction has made it to the Aya Blockchain by going to https://wmt-explorer.com/Testnet.

    Once there we need to look at the ‘Latest Transactions’ section and check to see if the 'MsgSetValAcc' Transaction we just sent has appeared on the Blockchain History.

    When we see it appear, we can then move on to the next step.

13. Now we need submit our 2nd of 2 Transactions to complete formal Registration to be a part of the Cardano Chain Follower root-list.

    We do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ayad tx chainfollower send-root --home /opt/aya --from <account> --chain-id aya_preview_501 -y
    {{< /highlight>}}

    > Note: After entering this command we will once again be asked to enter the Spending Password we set up earlier in the previous guide used specifically to set up our Validator Account. We must enter it now.
    >
    > Also, remember to replace <account> with the output of the previously completed cat command used to get this information. Also ensure to remove the **<>s** from the placeholder text above.

    If the Transcation has been successful we should see an output that contains a txhash value that consists of a string of capital letters and numbers, along with a lot of other information, that looks something like this

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    code: 0
    codespace: ""
    data: ""
    events: []
    gas_used: "0"
    gas_wanted: "0"
    height: "0"
    info: ""
    logs: []
    raw_log: '[]'
    timestamp: ""
    tx: null
    txhash: <Transaction Hash containing capital letters and numbers>
    {{< /highlight>}}

14. Before proceeding to the next step, we need to wait long enough so that the 2nd Registration step fully completed as expected. We are waiting for the Transaction that was just posted to the Aya Blockchain to make its way to a fully Minted Block, and to form a part of that Block's list of Transactions. 

    We can look to see if and when this Transaction has made it to the Aya Blockchain by going to https://wmt-explorer.com/Testnet.

    Once there we need to look at the ‘Latest Transactions’ section and check to see if the 'MsgSendRoot' Transaction we just sent has appeared on the Blockchain History.

15. Now we are going to complete the neccesary steps to ensure that our Validator is now actually a part of the Registered list of Cardano Chain Followers. 

    We will start by setting up our Validator's operator address as a variable for future steps.

    We do this by entering the following command

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    operator_address=$(ayad tendermint show-address --home /opt/aya)
    {{< /highlight>}}

16. Next we use that created variable to check to see if our newly Registered Chain Follower is in the list of Chain Follower roots.

    We do this by entering the following command
    
    {{< highlight bash "linenos=table,style=witchhazel" >}}
    ayad query chainfollower list-root | grep $operator_address
    {{< /highlight>}}
   
    We are now checking to see if entering this command returns back a result that looks like the following, with the **ayavalcons1** part of the response appearing fully in red. 

    {{< highlight bash "linenos=table,style=witchhazel" >}}
    val_addr: ayavalcons1<various lowercase letters and numbers>
    {{< /highlight>}}

If it does, **Congratulations!** We have now successfully completed setting up our Validator to be a full Chain Follwer root-list!




---
categories: ["Testnet"]
tags: ["setup", "install", "cosmovisor", "ayad"]
title: "Jailing"
linkTitle: "Jailing"
date: 2023-02-18
weight: 40
description: >
  What is jailing and how can I unjail?
---

>TL;DR: If a validator looses sync with the blockchain, they are punished (slashed) and delegator rewards are reduced.
 This is an incentive for the validator to monitor and maintaining their nodes on good infrastructure.

## What is Jailing?

Validators are the active actors that ensure the integrity and security of the network. 
Their tasks consist — in a nutshell — of validating the transactions, voting on the next state of the blockchain and committing it when a consensus is reached.

To maintain a high level of security and performance on the network, a validator is supposed to keep signing and committing blocks permanently. 
However, a validator can fail to commit blocks due to multiple reasons, such as a connection loss or a server failure. 
To protect the network, and prevent any performance drop, an inactive validator needs to be eliminated from the validator list. In Cosmos based chains this process of temporarily eliminating a validator is called “Jailing”.

## When am I Jailed?
A validator is jailed when they make **liveness or Byzantine fault**, when a validator is jailed, it will no longer be considered as an active validator until they are un-jailed. Furthermore, it cannot be un-jailed before downtime_jail_duration. This downtime_jail_duration is a network parameter which can be configured during genesis.

>Important:
    When a validator is jailed because of a byzantine fault, their validator public key is added to a list of permanently banned validators and cannot re-join the network as a validator with the same public key, see staking tombstone

## Network Parameters

The consensus params define the conditions for Jailing. 

- **signed_blocks_window**: Number of blocks for which the liveness is calculated for uptime tracking
- **min_signed_per_window**: Maximum percentage of blocks with faulty/missed validations allowed for an account in last; signed_blocks_window blocks before it gets deactivated;
- **downtime_jail_duration**: Duration for Jailing
- **slash_fraction_double_sign**: Percentage of funds being slashed when validator makes a byzantine fault
- **slash_fraction_downtime** : Percentage of funds being slashed when a validator is non-live.


## Network Parameters Aya
These parameters are in the genesis of the Aya-Sidechain.
{{< highlight go "linenos=table,style=witchhazel" >}}
signed_blocks_window : 100
min_signed_per_window : 0.5
downtime_jail_duration :  600s
slash_fraction_double_sign: 0.05
slash_fraction_downtime: 0.01
{{< /highlight>}}

## Slashing Mechanism: 
Punishments for a validator are triggered when they either make a byzantine fault or become non-live.

### Liveness Faults

A validator is said to be non-live when they fail to sign at least **min_signed_per_window blocks **(in percentage) in the last signed_blocks_window blocks successfully. signed_blocks_window and min_signed_per_window are network parameters and can be configured during genesis and can be updated during runtime by the governance module.

    For example, if block_signing_window is 100 blocks and min_signed_per_window is 0.5, 
    validator will be marked as non-live and jailed
    if they fail to successfully sign at least 100*0.5=50 blocks
    in last 100 blocks.


When a validator fails to successfully sign missed_block_threshold blocks in last block_signing_window blocks, it is immediately jailed and punished by deducting funds from their bonded and unbonded amount and removing them from active validator set. The funds to be deducted are calculated based on slash_fraction_downtime. 

### Byzantine Faults

A validator is said to make a byzantine fault when they sign conflicting messages/blocks at the same height and round. Tendermint has mechanisms to publish evidence of validators that signed conflicting votes so they can be punished by the slashing module. For example:

Validator who votes for two different blocks within a single round ("Equivocation validator"/ "Double signing");

Validator who signs commit messages for arbitrary application state ( "Lunatic validator").

>The evidence of a set of validators attempting to mislead a light client can also be detected and captured. However, even the Amnesia attack can be detected, punishment can not be applied at this stage, as we can not deduce the malicious validators.

 

## Un-jailing
When a jailed validator wishes to resume normal operations (after downtime_jail_duration has passed), they can create anunjail transaction which marks them as un-jailed. Validator will then rejoin the validator set once it has been successful un-jailed


>If the slashing mechanism is reducing your stake/voting power to zero you cannot unjail. In this case it is needed to increase your unslashed stake to regain voting power 

### Commands to unjail
{{< highlight go "linenos=table,style=witchhazel" >}}
ayad tx slashing unjail --from <aliasOfYourAccount> --home /opt/aya
{{< /highlight>}}

>aliasOfYourAccount is the account name you used when installing the validator. The -a argument.

If you cannot unjail due to zero stake you first need to self-delegate the additional token we sent to you before performing the unjailing
{{< highlight go "linenos=table,style=witchhazel" >}}
ayad tx staking delegate <validatorAddress(the ayavaloper one)> 1uswmt --home /opt/aya/ --from aliasOfYourAccount
{{< /highlight>}}

**How can I find my validator address?**

You can easily get the validator address using the Nodebase tool script showkeys.sh, located [here](/docs/tools/)


### Source

https://crypto.org/docs/chain-details/module_slashing.html#overview 
slashing module | Crypto.org Chain

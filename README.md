# Cosmos-based Node Ansible Setup

This repo is to set up the Cosmos-based node. It currently support:

- Akash
- BitCanna
- Chihuahua
- Comdex
- Cerberus
- Evmos
- Juno
- Kava
- KiChain
- Osmosis
- Sifchain
- Terra
- Umee

## Summary

You run one playbook and set up a node. For example:

```bash
ansible-playbook -i inventory juno.yml -e "target=juno_mainnet_main"
```

But before you rush with this easy setup, you probably want to read on so you understand the structure of this Ansible program and all the features it offers.

## Some Preparations

First, make sure that you have a production inventory file with your confidential server info. You will start by copying the sample inventory file (included in the repo). The sample file gives you a good idea on how to define the inventory.

```bash
cp inventory.sample inventory
```

Needless to say, you need to update the dummy values in the inventory file. For each node, you need to update the server IP, validator_name and log_name (log_name is optional if you do not install Promtail). While you are free to keep the default, you might also want to update:

1. ansible_user: The sample file assumes `ubuntu`, but feel free to use other user name. This user need sudo privilege.
1. ansible_port: The sample file assumes `22`. But if you are like me, you will have a different ssh port other than `22` to avoid port sniffing.
1. ansible_ssh_private_key_file: The sample file assumes `~/.ssh/id_rsa`, but you might have a different key location.
1. user_dir: The user's home directory. In the sample inventory file this is a computed variable based on the ansible_user. It assumes that it is not a root user and its home directory is `/home/{{ansible_user}}`
1. path: This is to make sure that the ansible_user can access the `go` executable. If your ansible_user is not `ubuntu`, you need to update it.
1. node_exporter: Default is `true`. Change it to `false` if you do not want to install node_exporter
1. promtail: Default is `true`. Change it to `false` if you do not want to install promtail
1. log_monitor: Enter your monitor server IP. It is most likely a private IP address if you use a firewall around your private virtual cloud (VPC). You do not need this if you do not install promtail

It is beyond the scope of this guide to help you create a sudo user, alternate ssh port, create a private key, install Ansible on your machine, etc. You can do a quick online search and find the answers. In my experience, Digital Ocean have some quality guides on these topics. Stack Overflow can help you trouble-shoot if you are stuck.

## Basic Cluster Structure

The basic cluster structure is:

1. Name each juno node as `juno_mainnet_main`, `juno_mainnet_backup`, etc. Group all Juno nodes into `juno_mainnet` group.
2. Name each juno node as `comdex_mainnet_main`, `Comdex_mainnet_test`, etc. Group all comdex nodes into `comdex_mainnet` group.
3. ...

The structure allows you to target `vars` to each node, or a group cluster, or the whole cluster. Make sure that you are familiar with the files in the `group_vars` folder. They follow this clustered structure closely.

## Playbooks

The key Ansible playbook is `<chain>.yml` files. It will set up a fresh node from scratch. For example:

```bash
ansible-playbook -i inventory akash.yml -e "target=akash_mainnet_main"
ansible-playbook -i inventory bitcanna.yml -e "target=bitcanna_mainnet_main"
ansible-playbook -i inventory chihuahua.yml -e "target=chihuahua_mainnet_main"
ansible-playbook -i inventory comdex.yml -e "target=comdex_mainnet_main"
ansible-playbook -i inventory cerberus.yml -e "target=cerberus_mainnet_main"
ansible-playbook -i inventory evmos.yml -e "target=evmos_mainnet_main"
ansible-playbook -i inventory juno.yml -e "target=juno_mainnet_main"
ansible-playbook -i inventory kava.yml -e "target=kava_mainnet_main"
ansible-playbook -i inventory kichain.yml -e "target=kichain_mainnet_main"
ansible-playbook -i inventory osmosis.yml -e "target=osmosis_mainnet_main"
ansible-playbook -i inventory sifchain.yml -e "target=sifchain_betanet_main"
ansible-playbook -i inventory terra.yml -e "target=terra_mainnet_main"
ansible-playbook -i inventory umee.yml -e "target=umee_mainnet_main"
```

If you prefer to install the node manually, you can run a 'prepare' playbook to set up a server for a cosmos-based chain without installing the node itself.

Playbooks are:

| Playbook             | Description                                                                                 |
| -------------------- | ------------------------------------------------------------------------------------------- |
| `prepare.yml `       | Prepare the server with node_exporter, promtail, go, cosmovisor, and firewall rules         |
| `auto_compound.yml ` | Copy an auto-compound shell script (more details below)                                     |
| `bitcanna.yml`       | Set up Bitcanna node. It includes the general `prepare` task and `bitcanna` specific task   |
| `chihuahua.yml`      | Set up Chihuahua node. It includes the general `prepare` task and `chihuahua` specific task |
| `comdex.yml`         | Set up Comdex node. It includes the general `prepare` task and `comdex` specific task       |
| `Cerberus.yml`       | Set up Cerberus node. It includes the general `prepare` task and `cerberus` specific task   |
| `evmos.yml`          | Set up Comdex node. It includes the general `prepare` task and `evmos` specific task        |
| `juno.yml`           | Set up Juno node. It includes the general `prepare` task and `juno` specific task           |
| `kava.yml`           | Set up Kava node. It includes the general `prepare` task and `kava` specific task           |
| `kichain.yml`        | Set up KiChain node. It includes the general `prepare` task and `kichain` specific task     |
| `osmosis.yml`        | Set up Osmosis node. It includes the general `prepare` task and `osmosis` specific task     |
| `sifchain.yml`       | Set up Sifchain node. It includes the general `prepare` task and `sifchain` specific task   |
| `terra.yml`          | Set up Terra node. It includes the general `prepare` task and `terra` specific task         |
| `umee.yml`           | Set up Umee node. It includes the general `prepare` task and `umee` specific task           |

## Auto Compound

The playbook will copy an auto-compound script to the user home directory.

| Playbook                            | Description                                                  |
| ----------------------------------- | ------------------------------------------------------------ |
| `auto_compound.yml`                 | Use this playbook when auto-compounding with own validator   |
| `auto_compound_for_delegation.yml ` | Use this playbook when auto-compounding with other validator |

```bash
ansible-playbook -i inventory auto_compound.yml -e "target=chihuahua_main"
ansible-playbook -i inventory auto_compound_for_delegation.yml -e "target=chihuahua_main"
```

You can run the script on the node with the following:

```bash
./auto_compound.sh <KEY> <PASSWORD>
./auto_compound_for_delegation.sh <KEY> <PASSWORD>
```

Alternatively, you can add a cronjob. For example, this following cronjob will run the auto-compound script daily at midnight. In this case, make sure that your server is super secure, as you will expose your key password in the crontab. Adopt this strategy at your own risk.

```bash
0 0 * * * /bin/bash /home/<USER>/auto_compound.sh <KEY> <PASSWORD>
0 0 * * * /bin/bash /home/<USER>/auto_compound_for_delegation.sh <KEY> <PASSWORD>
```

## Node Snapshot

We also offer node snapshot service for the validator community. We run snapshot script on any networks where we run a backup node. You need to have awscli installed and configured.

| Playbook       | Description                                  |
| -------------- | -------------------------------------------- |
| `snapshot.yml` | Copy the snapshot shell script to the server |

```bash
ansible-playbook -i inventory snapshot.yml -e "target=chihuahua_main_backup"
```

You can run the script on the node with the following:

```bash
./snapshot.sh
```

Alternatively, you can add a cronjob. For example, this following cronjob will run the snapshot script at the midnight

```bash
0 0 * * * /bin/bash /home/<USER>/snapshot.sh
```

## Relayers

We offer some simple scripts to upload hermes config files to our Juno and Osmosis relayers hubs. It is designed for our internal consumptions only. We installed our relayer software (Hermes) manually. However, because we update our config files rather frequently, we have decided to automate this specific task. The Ansible script is probably not helpful to you, but you might like to adapt our Hermes config file itself. For this reason, we have decided to include the playbooks here.

```bash
ansible-playbook -i inventory relayer_juno.yml -e "target=relayer_juno"
ansible-playbook -i inventory relayer_osmosis.yml -e "target=relayer_osmosis"
```

## Horcrux

Our friend [coffeecoaster](https://github.com/coffeeroaster) has contributed this Horcrux playbook. Horcrux is a multi-party-computation (MPC) signing service for Tendermint nodes. Its github repo is [here](https://github.com/strangelove-ventures/horcrux). It is assumed that you already have a node with all the necessary toolings (Go, Comsovisor, jq, .profile setup, etc). If you install a tendermint node with one of our Ansible playbook, you have these toolings already.

```bash
ansible-playbook -i inventory horcrux.yml -e "target=juno_mainnet_main"
```

In case you do not have all the necessary toolings, then we recommend you run `prepare` playbook first.

```bash
ansible-playbook -i inventory prepare.yml -e "target=juno_mainnet_main"
ansible-playbook -i inventory horcrux.yml -e "target=juno_mainnet_main"
```

## Additional Info

1. Polkachu Operation Summary table: https://polkachu.com/summary_table
1. Snapshot service: https://polkachu.com/tendermint_snapshots
1. State-Sync service: https://polkachu.com/state_sync
1. [General](docs/general.md)
1. [Polkachu's Mainnet Setup](docs/mainnet.md)
1. [Polkachu's Testnet Setup](docs/testnets.md)
1. [Validator Server Migration Best Practice](docs/validator_server_migration_best_practice.md)

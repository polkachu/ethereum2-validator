# Polkdadot Validation Node Ansible Setup

This repo is to set up Polkadot Validation node. This repo is a heavily modified version of https://github.com/w3f/polkadot-validator-setup

## Set Up Kusama/Polkadot Validator

Make sure that you have a production inventory file. You can start by copying the sample inventory file (included in the repo) if you start from scratch. The sample file gives you a good idea about how to define the inventory.

```bash
cp inventory.sample inventory
```

Make sure that you are familiar with the files in group_vars. They often need to change to stay up to date with the latest release. It is very possible that you will get an error on checksum of data restore in your first attempt, as the data screenshot by polkashots.io is updated frequently.

The key validator ansible file is polkadot_full_setup.yml, which will set up a new fresh validator from scratch.

```bash
ansible-playbook -i inventory polkadot_full_setup.yml -e "target=VALIDATOR_TARGET"
```

VALIDATOR_TARGET could be a host (kusama1, kusama2, polkadot1, polkadot2, etc), could be a group (kusama, polkadot), or all validators (validators)

The most commonly used playbooks are:

| Playbook                | Description                                                                              |
| ----------------------- | ---------------------------------------------------------------------------------------- |
| polkadot_full_setup.yml | Run the initial full setup                                                               |
| polkadot_prepare.yml    | Do the prep work, such as firewall, set up proxy, copy service files, create users, etc. |
| polkadot_update.yml     | Update the polkadot binary and restart service. You probably need to use it regularly    |
| polkadot_restore.yml    | Restore the polkadot database with screenshot. Only useful for initial setup             |
| node_exporter.yml       | Update Node Exporter                                                                     |
| process_exporter.yml    | Update Process Exporter                                                                  |

The less commonly used playbooks are:

| Playbook                     | Description                                                                  |
| ---------------------------- | ---------------------------------------------------------------------------- |
| polkadot_backup_keystore.yml | Backup keystore (Not sure about use case)                                    |
| polkadot_clean_logs.yml      | Clean journal logs (Probably useful when disk is full)                       |
| polkadot_restart.yml         | Restart polkadot ad hoc (Probably useful when server runs hot for no reason) |
| polkadot_stop.yml            | Stop polkadot ad hoc                                                         |
| polkadot_rorate_keys.yml     | Rotate session keys the easy way                                             |

## Set Up Monitor

The key monitor ansible file is monitor_full_setup.yml, which will set up a new fresh monitor from scratch. It will set up firewall, install Prometheus, Grafana and Alert Manager

```bash
ansible-playbook -i inventory monitor_full_setup.yml
```

You might want to run separate playboos as needed:

| Playbook                  | Description                           |
| ------------------------- | ------------------------------------- |
| monitor_full_setup.yml    | Full Setup                            |
| monitor_prepare.yml       | Some prep work; mostly firewall stuff |
| monitor_prometheus.yml    | Update Prometheus                     |
| monitor_grafana.yml       | Update Grafana                        |
| monitor_alert_manager.yml | Update Alert Manager                  |

For now, you will most likely need to edit monitor_prometheus/templates/prometheus.yml.j2 file to update "targets". For now, it is a little manual. Later I might automate it based on the main inventory file.

## Set Up Ethereum Validator

Setting up Ethereum validator is mostly automated, but you still need to generate some keys in the middle of the setup.

Step 1: Do the setup.

```bash
ansible-playbook -i inventory ethereum_1_pre_deposit.yml -e "target=VALIDATOR_TARGET"
```

This will:

1. setup firewall
2. install eth1 client (Geth)
3. install node exporter
4. install process exporter
5. install deposit client (used to manage deposit keys)
6. install eth2 lighthouse client

Step 2: Make the deposit according to the Medium post

1. Make deposit
2. Run command: lighthouse --network pyrmont account validator import --directory $HOME/<PathToValidatorKeys> --datadir /var/lib/lighthouse
3. Type in the password
4. sudo chown root:root /var/lib/lighthouse

Now you are done with setting up the wallet

Helpful Eth2 log file monitoring:

1. sudo journalctl -fu geth.service
2. sudo journalctl -fu lighthousebeacon.service
3. sudo journalctl -fu lighthousevalidator.service
4. sudo tail /var/log/syslog -n 100 | grep "head slot" (helpful when the beacon chain is behind)

## Update All Servers

Often, you might want to update all servers. That's easy. Just run:

```bash
ansible-playbook -i inventory all_apt_update.yml
```

### table to details all the other features.

## TODO

A script to check the latest release.

make sure to use the extra varraible to set target

## TODO

Make a shell script that accept a target varaible and it will set everything up as a validators

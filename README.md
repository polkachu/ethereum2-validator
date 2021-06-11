# Ethereum 2 Validator Setup With Lighthouse Client

This repo is to set up Ethereum 2 Validator with Lighthouse Client. The repo is based on https://someresat.medium.com/guide-to-staking-on-ethereum-2-0-ubuntu-pyrmont-lighthouse-a634d3b87393.

Setting up Ethereum validator is mostly automated, but you still need to generate deposit keys and deposit your ETH in the process. As a result, there are two ansible playbooks: one for before the deposit key generation and the other after.

## Step 0: Make your own invetory file.

Get started with

```
cp inventory.sample inventory
```

## Step 1: Setup before deposit keys

Run the first playbook. Make sure that you have adjusted your inventory file. Replace VALIDATOR_TARGET with your host or group.

```bash
ansible-playbook -i inventory ethereum_step_1.yml -e "target=VALIDATOR_TARGET"
```

This will:

1. set up firewall
2. install node exporter
3. install process exporter
4. install eth2 lighthouse client

It is assumed that you will get the eth1 client from a third-party such as Infura. A free account should be sufficient.

## Step 2: Make deposit keys and associate the keys with Lighthouse

1. Download the deposit key generator from https://github.com/ethereum/eth2.0-deposit-cli/releases. And generate keys. All security measures apply here.

```
./deposit new-mnemonic --num_validators <numberofvalidators> --mnemonic_language=english --chain pyrmont
```

2. Copy the key files to the validator server using command like this. I assume the ubuntu user here:

```
scp -r -P 22 ~/local-path/validator_keys user@remote-server:/home/ubuntu
```

3. On the remote server, run command:

```
lighthouse --network {{ pyrmont or mainnet }} account validator import --directory /home/ubuntu/validator_keys --datadir /var/lib/lighthouse
```

## Step 3: Set up Lighthouse Beacon node and validator

Run the first playbook. Make sure that you have adjusted your inventory file. Replace VALIDATOR_TARGET with your host or group.

```bash
ansible-playbook -i inventory ethereum_step_2.yml -e "target=VALIDATOR_TARGET"
```

This will:

1. Install Lighthouse Beacon client
2. install Lighthouse validator

Helpful Eth2 log file monitoring to make sure everything is working. You might need to wait for a few days before everything is fully synced. Good news is that you have not deposited Eth, so just relex if you find bugs.

```
sudo journalctl -fu lighthousebeacon.service
sudo journalctl -fu lighthousevalidator.service
sudo tail /var/log/syslog -n 100
geth attach http://127.0.0.1:8545
eth.syncing
net.peerCount
```

## Step 4: Make deposit and see it go!

Triple check all key and password backups.
Reboot your machine and make sure the services come back up.
Understand how to update the client and server software.
Use htop to monitor resources on the local machine.
Get familiar with beaconcha.in so you can monitor your validators. They offer alerting (via email — sign up required) and up to 3 POAPs.
Join the Ethstaker and Lighthouse Discord for important notifications.

## Extra Steps: Update Geth

```
sudo systemctl stop lighthousevalidator
sudo systemctl stop lighthousebeacon
sudo systemctl stop geth
sudo apt update && sudo apt upgrade
sudo systemctl start geth
sudo systemctl status geth # <-- Check for errors
sudo journalctl -fu geth # <-- Monitor
sudo systemctl start lighthousebeacon
sudo systemctl status lighthousebeacon # <-- Check for errors
sudo journalctl -fu lighthousebeacon # <-- Monitor
sudo systemctl start lighthousevalidator
sudo systemctl status lighthousevalidator # <-- Check for errors
sudo journalctl -fu lighthousevalidator # <-- Monitor
```

## Extra Steps: Update Lighthouse

Update the group_vars file and run the playbook

```bash
ansible-playbook -i inventory update_lighthouse.yml -e "target=VALIDATOR_TARGET"
```

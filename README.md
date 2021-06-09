# Ethereum 2 Validator Setup With Lighthouse Client

This repo is to set up Ethereum 2 Validator with Lighthouse Client. The repo is based on https://someresat.medium.com/guide-to-staking-on-ethereum-2-0-ubuntu-pyrmont-lighthouse-a634d3b87393.

Setting up Ethereum validator is mostly automated, but you still need to generate deposit keys and deposit your ETH in the process. As a result, there are two ansible playbooks: one for before the deposit key generation and the other after.

## Step 0: Make your own invetory file.

Get started with

```
cp inventory.sample inventory
```

## Step 1: Setup before deposit keys

```bash
ansible-playbook -i inventory ethereum_step_1.yml -e "target=VALIDATOR_TARGET"
```

This will:

1. set up firewall
2. install eth1 client (Geth)
3. install node exporter
4. install process exporter
5. install eth2 lighthouse client

## Step 2: Make deposit keys and associate the keys with Lighthouse

1. Download the deposit key generator from https://github.com/ethereum/eth2.0-deposit-cli/releases. And generate keys. All security measures apply here.

```
./deposit new-mnemonic --num_validators <numberofvalidators> --mnemonic_language=english --chain pyrmont
```

2. Copy the key files to the validator server using command like this. I assume the ubuntu user here:

```
scp -r -P 22 ~/local-path/validator_keys remote-server:/home/ubuntu
```

3. On the remote server, run command:

```
lighthouse --network {{ pyrmont or mainnet }} account validator import --directory /home/ubuntu/validator_keys --datadir /var/lib/lighthouse
```

## Step 3: Set up Lighthouse Beacon node and validator

```bash
ansible-playbook -i inventory ethereum_step_2.yml -e "target=VALIDATOR_TARGET"
```

Helpful Eth2 log file monitoring:

```
sudo journalctl -fu geth.service
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
Share any feedback for this guide on Discord, Twitter, or Reddit.
Help others with their setup on the Ethstaker discord.
Share this guide using the friend link!
Tips: somer.eth

## Extra Steps:

update geth
update lighthouse

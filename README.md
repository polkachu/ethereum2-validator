# Ethereum 2 Validator Setup With Lighthouse Client

This repo is to set up Ethereum 2 Validator with Lighthouse Client. The repo is based on https://someresat.medium.com/guide-to-staking-on-ethereum-2-0-ubuntu-pyrmont-lighthouse-a634d3b87393.

Setting up Ethereum validator is mostly automated, but you still need to generate deposit keys and deposit your ETH in the process. As a result, there are two ansible playbooks: one for before the deposit and the other after.

d
d
d
d
d
d
d
d
d

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

See the latest block: https://goerli.etherscan.io/blocks

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

Testnet
./deposit new-mnemonic --num_validators <numberofvalidators> --mnemonic_language=english --chain pyrmont

Mainnet
./deposit new-mnemonic --num_validators <NumberOfValidators> --mnemonic_language=english --chain mainnet

Extra steps:

1. Deposit ETH
2. Copy files to the remote folder: scp -r -P 2200 ~/Documents/crypto/ethereum/validator_keys ubuntu@143.198.225.90:/home/ubuntu
3. Run shell command to tie the validation key:
   lighthouse --network pyrmont account validator import --directory /home/ubuntu/validator_keys --datadir /var/lib/lighthouse

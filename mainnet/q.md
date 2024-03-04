---
title: 🇶 Q Blockchain
---
# Q Blockchain


## Basic Configuration

Clone the repository

```bash
$ git clone https://gitlab.com/q-dev/mainnet-public-tools
```
Windows (if you don`t have git installed):

```
# Download the contents of the Git repository
Invoke-WebRequest -Uri https://gitlab.com/q-dev/mainnet-public-tools/-/archive/master/mainnet-public-tools-master.zip -OutFile mainnet-public-tools-master.zip

# Extract the contents of the ZIP file
Expand-Archive -Path mainnet-public-tools-master.zip -DestinationPath .

# Remove the ZIP file
Remove-Item -Path mainnet-public-tools-master.zip
```


and go to the `/validator` directory

Linux, macOS, other Unix-like systems:

```bash
$ cd mainnet-public-tools/validator
```
Windows:
```
Set-Location -Path "mainnet-public-tools\validator"
```


This directory contains the `docker-compose.yaml` file for quick launching of the validator node with preconfigurations on rpc, blockchain explorer using `.env` (which can be created from `.env.example`).

## Generate a Keypair for Validator

Copy `.env.example` to `.env` and in `/validator` directory:

Linux, macOS, other Unix-like systems:
```
cp .env.example .env
```


Windows:
```
# This will copy the .env.example file to a new file named .env.
Copy-Item -Path ".\env.example" -Destination ".\env"

```

In order to sign blocks and receive reward, a validator needs a keypair.
Create a `/keystore` directory, then a password which will be used for private key encryption and save it into a text file `pwd.txt` in `/keystore` directory.
Assuming you are in `/validator` directory, issue this command in order to generate a keypair:  

```bash
$ docker run --entrypoint="" --rm -v $PWD:/data -it qblockchain/q-client:v1.3.5 geth account new --datadir=/data --password=/data/keystore/pwd.txt

```

The output of this command should look like this:

```text
Your new key was generated

Public address of the key:   0xb3FF24F818b0ff6Cc50de951bcB8f86b52287dac
Path of the secret key file: /data/keystore/UTC--2021-01-18T11-36-28.705754426Z--b3ff24f818b0ff6cc50de951bcb8f86b52287dac

- You can share your public address with anyone. Others need it to interact with you.
- You must NEVER share the secret key with anyone! The key controls access to your funds!
- You must BACKUP your key file! Without the key, it's impossible to access account funds!
- You must REMEMBER your password! Without the password, it's impossible to decrypt the key!
```

This way, a new private key is generated and stored in keystore directory encrypted with password from pwd.txt file. In our example, *0xb3FF24F818b0ff6Cc50de951bcB8f86b52287DAc* (**you will have a different value**) is the address corresponding to the newly generated private key.

Alternatively, you can generate a secret key pair and according file on this [page](https://vanity-eth.tk/) and save it to the `/keystore` directory manually.
Also you may use `create-geth-private-key.js` script in `/js-tools` folder.

Whether you chose to provide your own vanity keys or use the above command to create a keypair, please ensure that the directory `/keystore` contains the following files:

```text
validator
|   ...
|   ...
└ keystore
  |   UTC--2021-01-18T11-36-28.705754426Z--b3ff24f818b0ff6cc50de951bcb8f86b52287dac
  |   pwd.txt
```

> **Note: ** *Following our example, pwd.txt contains the password to encrypted file "UTC--2021-01-18T11-36-28.705754426Z--b3ff24f818b0ff6cc50de951bcb8f86b52287dac" in clear text.*

If you want to change the password in the future, you need to stop the node first.

```bash
$ docker-compose down
```

Then start password reset procedure with

```bash
$ docker-compose run validator-node --datadir /data account update 0xb3ff24f818b0ff6cc50de951bcb8f86b52287dac
```

> **Note: ** *You need to remove address _0xb3ff24f818b0ff6cc50de951bcb8f86b52287dac_ and add your account address instead.*

## Get Q Tokens

In order to become a validator, you will need to put some stake in validators contract, so you need Q tokens for this. If you do not have Q tokens already, you can purchase them directly from Q Development AG. Follow [this link](https://q.org/why-q#why-q-validators-block) to learn more.

For getting started operationaly, you can claim small amounts of Q tokens from the [Mainnet Faucet](https://faucet.q.org). You can find more information on the faucet and on how to use it in the [Faucet documentation](how-to-claim-q-tokens.md).

## Configure Setup

Edit the environment file in `/validator` directory:

Linux, macOS, other Unix-like systems:

```bash
$ nano .env
```

Windows:
```
#This will open the .env file in Notepad for editing. If you prefer to use a different text editor, replace notepad.exe with the appropriate command for your editor.
notepad.exe .\env
```


Put your address without leading 0x from the step 3, into `ADDRESS`, your public IP address (please make sure your machine is reachable at the corresponding IP) into `IP` (this is required for discoverability by other network participants) and optionally choose a port for p2p protocol (or just leave default value). The resulting `.env` file should look like this:

```text
# docker image for q client
QCLIENT_IMAGE=qblockchain/q-client:v1.3.8

# your q address here (without leading 0x)
ADDRESS=b3FF24F818b0ff6Cc50de951bcB8f86b52287DAc

# your public IP address here
IP=193.19.228.94

# the port you want to use for p2p communication (default is 30303)
EXT_PORT=30303

# extra bootnode you want to use
BOOTNODE1_ADDR=enode://22adab037308f02abbb0fd7e831c75afa367b36615b2a0358a5c4673912cf384de6c8e688371822488622ebee383aeea5d41087160cb70484a9f1671876871b1@bootnode.q.org:30301
BOOTNODE2_ADDR=enode://3021f73a6f14f8594384923f7f0228f81a806d1708e5c046db12661bdce6b0f10625fae12771aa36f7a4d1f110d4e5a589bf3d34ec4b1d2c6d10e382d90f6983@extrabootnode.q.org:30314
BOOTNODE3_ADDR=enode://34b9e4e18bc37e4437bc0a9b10ac8ae5d0aab2b2e827310e90ec1012e818d07962b162d98e083ec5487e0cf87d1ffefb46332ec05209ec82fb675ae7afe3e241@extrabootnode.q.org:30315
```

Next, you need to edit `config.json` as this file is required for staking. Put your address from above into the address field and password from `/keystore/pwd.txt` into the password field. Resulting `config.json` should be similar to this:

```json
    {
      "address": "b3FF24F818b0ff6Cc50de951bcB8f86b52287DAc",
      "password": "supersecurepassword",
      "keystoreDirectory": "/data",
      "rpc": "https://rpc.q.org"
    }
```

## Put Stake in Validators Contract

As was mentioned previously, you need to put stake to validators contract in order to become a validator. 

You can use the dApp "Your HQ" that can be found at [https://hq.q.org](https://hq.q.org). Ultimately, you need to `Join Validator Ranking` to receive rewards. The according functionality is located at `Staking -> Validator Staking`.

## Add your Validator to https://stats.q.org

If you want your validator to report to the [network statistics](https://stats.q.org), you can add an additional flag to the node entrypoint within file `/validator/docker-compose.yaml`, it should look like this:

```yaml
validator-node:
  image: $QCLIENT_IMAGE
  entrypoint: ["geth", "--ethstats=<Your_Validator_Name>:<Mainnet_access_key>@stats.q.org", "--datadir=/data", ...]
```

`<Your_Validator_Name>` can be chosen arbitrarily. It will be displayed in the statistics. If you want to disclose your ID, this could be something like "OurCoolCompany - Don't trust, verify". You can use special characters, emojis as well as spaces. If you prefer to stay anonymous, we would appreciate to include the beginning of your validator Q address, so there is a link between your client and your address.

In order to find out the `<Mainnet_access_key>` please write us [on Discord](https://discord.gg/YTgkvJvZGD).

## Launch Validator Node

Now launch your validator node using docker-compose file in validator directory:

```bash
$ docker-compose up -d
```

Note: Check our nodes real-time logs with the following command:

```bash
$ docker-compose logs -f --tail "100"
```

## Find additional peers

In case your client can't connect with the default configuration, we recommend that you add an additional flag referring to one of our additional peers (`$BOOTNODE1_ADDR`, `$BOOTNODE2_ADDR`or `$BOOTNODE3_ADDR`) within `docker-compose.yaml` file:

```yaml
validator-node:
  image: $QCLIENT_IMAGE
  entrypoint: ["geth", "--bootnodes=$BOOTNODE1_ADDR,$BOOTNODE2_ADDR,$BOOTNODE3_ADDR", "--datadir=/data", ...]
```

## Verify that Node is producing Blocks

In order for you to start validating, you must wait for the new epoch (i.e. validation cycle). If everything went correctly before and the committed stake was sufficient to enter the validator shortlist, your validator node will start to produce blocks in the next validation cycle.
Please note that upon start you are likely to see a lot of warnings in q-client logs:

```text
WARN [01-18|13:12:00.431] Block sealing failed          err="unauthorized signer"
```

This is actually ok, as the node needs some time to synchronize with the peers of Q network. Until a full sync is reached, it may happen that your node already starts block creation using the most recent snapshot in which you are the only validator. After successful peer discovery, there warnings will disappear.

  > **Note: ** *All validators are required to run an omnibridge-oracle as per Constitution. Please see [here](how-to-setup-omnibridge.md#Configure-OmniBridge-oracle) for a tutorial how to do this.*

## Exit the Validator Ranking

If you want to exit the Validator Ranking, you must `Announce Withdrawal` within `Staking -> Validator Staking -> Validator Manage` of 100% of your self-staked Q token. After doing that, you will be taken out of the ranking immediately, though your node might still validate blocks until the next validation cycle begins (within max. 8 minutes).

The announced amount will be put on an escrow balance for a certain time (see constitution parameter `constitution.valWithdrawP`) until it can be withdrawn fully. Re-joining the panel is possible any time by putting back stake or reducing the announced withdrawal amount.

  > **Note: ** *A temporary exit from the ranking is possible as described above. For re-entering, you need to announce withdrawal of `0` Q which overwrites your initial announcement and restores your self-stake to 100%. You need to `Join Validator Ranking` again to finalise the re-entering procedure. A temporary exit (or pause) might be useful if you are planning a maintenance downtime of your node for example.*

## Updating Q-Client & Docker Images

To upgrade the node follow the instructions [Upgrade Node](how-to-upgrade-node.md)

![image](https://user-images.githubusercontent.com/47001602/180651661-ad6611e3-cf82-4246-9cc1-454952c177c7.png)

# How to set up a NEAR validator node on shardnet using the top cloud providers

So, you want to participate in Stake Wars and earn rewards for helping decentralise and secure the NEAR ecosystem? Then you're in the right place! However, unless you have tons of spare bare metal lying around, you'll need to rent your own servers. 

Popular cloud providers include:

- Amazon Web Services
- Google Cloud Platform
- Microsoft Azure
- IBM Cloud
- DigitalOcean
- Hetzner

Here, we'll cover the first (AWS), since it's the biggest.

There are roughly 4 steps required:

## 1/4. Create your Shardnet wallet

Wallets in the Stake Wars world are different. We need a special one to participate in this battle. Head over to https://wallet.shardnet.near.org/ and create yours!
On creating the wallet, you'll immediately be credited with test NEAR. Here's what it will look like:


<img width="829" alt="Screenshot 2022-07-24 at 15 31 15" src="https://user-images.githubusercontent.com/47001602/180651776-44386f77-d267-49b1-bb59-3deb516e6b1b.png">

## 2/4: Set up your server
Next up, we need a machine! Here are the specs you'll need 👇


| Hardware       | Chunk-Only Producer  Specifications                                   |
| -------------- | ---------------------------------------------------------------       |
| CPU            | 4-Core CPU with AVX support                                           |
| RAM            | 8GB DDR4                                                              |
| Storage        | 500GB SSD                                                             |


Not too bad, eh? Let's head over to AWS and see how to do this...

First, head over to the 'Instances tab'

<img width="237" alt="Screenshot 2022-07-24 at 15 34 12" src="https://user-images.githubusercontent.com/47001602/180651936-b7f06825-dcaa-45e4-8eeb-b82f5e7dfbd2.png">

Then, let's launch an instance by smashing this button: 
<img width="224" alt="Screenshot 2022-07-24 at 15 34 37" src="https://user-images.githubusercontent.com/47001602/180651949-fe8784ce-d6df-4db5-976d-1e2433c2c59c.png">

For this, we're going to be using Ubuntu, so let's grab that:

<img width="511" alt="Screenshot 2022-07-24 at 15 35 05" src="https://user-images.githubusercontent.com/47001602/180651962-4513a99e-8917-4004-9df1-4eaeee335880.png">

Next, we'll need hardware that satisfies the server requirements, so let's get ourselves the t2.xlarge - nice!

<img width="505" alt="Screenshot 2022-07-24 at 15 35 57" src="https://user-images.githubusercontent.com/47001602/180651985-ccf4a722-d7b2-461e-9d60-890d502872ec.png">

Finally, we need storage! We'll go ahead and give ourselves the 500GB required:
<img width="750" alt="Screenshot 2022-07-24 at 15 43 11" src="https://user-images.githubusercontent.com/47001602/180652314-00dc5f64-bef8-46ac-8ce1-2c315d0ef72a.png">

**Note on pricing** 💰

Ubuntu is free, and storage is relatively cheap - what will cost you is the compute (the t2.xlarge costs $0.2112 / hr at the time of writing). Remember, this is going to be on 24/7, so that's about $5 per day, and or $152 per month!

See here for more info: https://aws.amazon.com/ec2/pricing/on-demand/

To save, make sure to sign up for AWS credits! These may be provided to new startups, or academic institutions and other parties. Also, you can purchase long-term contracts (called reserve instances) so save if you know you're going to be validating long-term! 

## 3/4: Install everything! 
Now, we need to install stuff on our empty server. Let's go!👇

All you need to do now is literally just copy/paste these steps into the terminal (see https://github.com/near/stakewars-iii/blob/main/challenges/002.md).


1. Install developer tools:
```
sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo
```
2. Install Python pip:

```
sudo apt install python3-pip
```
3. Set the configuration:

```
USER_BASE_BIN=$(python3 -m site --user-base)/bin
export PATH="$USER_BASE_BIN:$PATH"
```

4. Install Building env
```
sudo apt install clang build-essential make
```

5. Install Rust & Cargo
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

6. Press 1 and press enter.

7. Source the environment
```
source $HOME/.cargo/env
```

8. Clone `nearcore` project from GitHub

```
git clone https://github.com/near/nearcore
cd nearcore
git fetch
```

9. Checkout to the commit needed. Please refer to the commit defined in [this file](https://github.com/near/stakewars-iii/blob/main/commit.md). 
```
git checkout <commit>
```

10. Compile `nearcore` binary
In the `nearcore` folder run the following commands:

```
cargo build -p neard --release --features shardnet
```

11. Initialize working directory

In order to work properly, the NEAR node requires a working directory and a couple of configuration files. Generate the initial required working directory by running:

```
./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis
```


12. Replace the `config.json`

From the generated `config.json`, there two parameters to modify:
- `boot_nodes`: If you had not specify the boot nodes to use during init in Step 3, the generated `config.json` shows an empty array, so we will need to replace it with a full one specifying the boot nodes.
- `tracked_shards`: In the generated `config.json`, this field is an empty. You will have to replace it to `"tracked_shards": [0]`

```
rm ~/.near/config.json
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
```


13. Run the node
To start your node simply run the following command:

```
cd ~/nearcore
./target/release/neard --home ~/.near run
```

----



4/4: Connect up your wallet!

Final stretch! We now need to tell our server what's going on, and how to connect up to NEAR. 
First, install the NEAR-CLI and then run:

```
near login
```
This will bring up a link, which you should paste into your browser.

This link will bring up the following:

<img width="826" alt="Screenshot 2022-07-24 at 15 50 46" src="https://user-images.githubusercontent.com/47001602/180652674-01951a6a-a4a1-4393-a6ef-73d154e707f5.png">
Click 'Next', and proceed to authorise the transaction. Then, go back to the console. 
Now, enter your wallet address (xx.shardnet.near) and we're almost done!


First, create a `validator_key.json` file.

Then generate the key file:

```
near generate-key <pool_id>
```


Now, copy the file generated to the shardnet folder:
Make sure to replace <pool_id> by your accountId
```
cp ~/.near-credentials/shardnet/YOUR_WALLET.json ~/.near/validator_key.json
```
* Edit “account_id” => xx.factory.shardnet.near, where xx is your PoolName
* Change `private_key` to `secret_key`

> Note: The account_id must match the staking pool contract name or you will not be able to sign blocks.\

File content must be in the following pattern:
```
{
  "account_id": "xx.factory.shardnet.near",
  "public_key": "ed25519:HeaBJ3xLgvZacQWmEctTeUqyfSU4SDEnEwckWxd92W2G",
  "secret_key": "ed25519:****"
}
```

Now, we can start the validator node! 🥳

```
target/release/neard run
```
If you want, you can follow the instructions here to set up systemd: https://github.com/near/stakewars-iii/blob/main/challenges/002.md. This will ensure the server continues even after closing your console window.

## 5/4: Set up the Staking pool!

You thought we were done! But hey, this is blockchain! There are always a few more steps than you expect to get up and running. Finally, we need to make a staking pool. N

Run this command to make it:

```
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<pool id>", "owner_id": "<accountId>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<accountId>" --amount=30 --gas=300000000000000
```

NEAR has a great explanation of what the above placeholders do here:

* **Pool ID**: Staking pool name, the factory automatically adds its name to this parameter, creating {pool_id}.{staking_pool_factory}
Examples:   

- If pool id is stakewars will create : `stakewars.factory.shardnet.near`

* **Owner ID**: The SHARDNET account (i.e. stakewares.shardnet.near) that will manage the staking pool.
* **Public Key**: The public key in your **validator_key.json** file.
* **5**: The fee the pool will charge (e.g. in this case 5 over 100 is 5% of fees).
* **Account Id**: The SHARDNET account deploying and signing the mount tx.  Usually the same as the Owner ID.

After you run the above command, you'll see a link gets printed to the console. Open that in your browser and you'll see the following:
<img width="1441" alt="Screenshot 2022-07-24 at 15 57 24" src="https://user-images.githubusercontent.com/47001602/180652939-b42f0e18-06d8-4867-85d5-a5a1d9f99533.png">

Congrats!! You've now successfully set up your validator and staking pool! Welcome to the NEAR community! 🚀



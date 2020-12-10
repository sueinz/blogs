### Deploy Ghost Blog Via Akash Decloud

**Ghost**

>The easiest way to get a production instance deployed is with our official **[Ghost(Pro)](https://ghost.org/pricing/)** managed service. It takes about 2 minutes to launch a new site with worldwide CDN, backups, security and maintenance all done for you.
>
>For most people this ends up being the best value option cause of [how much time it saves](https://ghost.org/docs/concepts/hosting/) — and 100% of revenue goes to the Ghost Foundation; funding the maintenance and further development of the project itself. So you’ll be supporting open source software *and* getting a great service!
>
>If you prefer to run on your own infrastructure, we also offer official 1-off installs and managed support and maintenance plans via **[Ghost(Valet)](https://valet.ghost.org/)** - which can save a substantial amount of developer time and resources.

**Akash DeCloud**

> The DeCloud for DeFi, and the world's first decentralized cloud computing marketplace.

The whole tutorial is roughly divided into several parts: preparing docker image, preparing akash deployment environment, and deploying ghost.

#### Prepare docker image

We found the official ghost image from [hub.docker.com](https://hub.docker.com/_/ghost)

```
mysql
```

#### Akash deployment environment preparation (Edgenet testnet)

**Install wallet**

The environment used here is ubuntu18.04

1. Download [akash wallet](https://github.com/ovrclk/akash/releases/download/v0.9.0-rc13/akash_0.9.0-rc13_linux_arm64.zip) program

   ```
   wget https://github.com/ovrclk/akash/releases/download/v0.9.0-rc13/akash_0.9.0-rc13_linux_arm64.zip
   ```

2. Unzip

   ``` 
   tar -xvf akash_0.9.0-rc13_linux_arm64.zip
   ```

3. Rename directory

   ```
   mv akash_0.9.0-rc13_linux_arm64 akash
   ```

4. Configure environment variables

   `/work/akash/akash` for the path we just unzipped

   edit `/etc/profile` , add the following at the end

   ```
   export PATH=$PATH:/work/akash/akash
   ```

   run after

   ```
   akash version
   ```

   output version number

   ```
   v0.9.0-rc13
   ```

5. Create wallet address

   Here you need to prepare some variables in advance, `KEY_NAME` is a string set by yourself, the default is alice

   ```
   export KEY_NAME=alice
   export KEYRING_BACKEND=local
   ```

   enerate wallet address

   ```
   akash --keyring-backend "$KEYRING_BACKEND" keys add "$KEY_NAME"
   ```

   the output is as follows

   ```
   - name: alice
     type: local
     address: akash1cz87pqkad72gggrv3t7y2x9z56h9gqghlnx3j3
     pubkey: akashpub1addwnpepqtnydvj056gy64uuquldq5yx7mr8ncmn3ut59wwl9p83d8h2v4rtg5xa3vn
     mnemonic: ""
     threshold: 0
     pubkeys: []
   
   
   **Important** write this mnemonic phrase in a safe place. It is the only way to recover your account if you ever forget your password.
   
   town wolf margin parrot strong disease dance eyebrow inflict meadow crunch version tube elite interest movie uphold column shift fox excuse humble nest call
   ```

   >address is the account address we generated
   >
   >The last 24 words are very important. They are the mnemonic words of our wallet. We need to save them and restore them when importing them

**Receive test coins**

1. Set a few commonly used variables first

   ```
   Set a few commonly used variables firstexport AKASH_NET="https://raw.githubusercontent.com/ovrclk/net/master/edgenet"
   export AKASH_CHAIN_ID="$(curl -s "$AKASH_NET/chain-id.txt")"
   exprot AKASH_NODE="$(curl -s "$AKASH_NET/rpc-nodes.txt" | shuf -n 1)"
   export ACCOUNT_ADDRESS=akash1cz87pqkad72gggrv3t7y2x9z56h9gqSet a few commonly used variables firstghlnx3j3
   ```

   >AKASH_NET: the base URL of the network you want to connect to
   >
   >AKASH_CHAIN_ID: the ID of the chain to be connected
   >
   >AKASH_NODE: node RPC address
   >
   >ACCOUNT_ADDRESS: the wallet address generated earlier
2. Run the command to view the faucet address for receiving test coins

   ```
   curl "$AKASH_NET/faucet-url.txt"
   ```

   the output is as follows

   ```
   https://akash.vitwit.com/faucet
   ```

3. Open this address in the browser and fill in the wallet address after passing the man-machine authentication. After waiting for a while, the account receives the test currency, and you can check the balance through the command

   ```
   akash --node "$AKASH_NODE" query bank balances "$ACCOUNT_ADDRESS"
   ```

   the output is as follows

   ```
   balances:
   - amount: "100000000"
     denom: uakt
   pagination:
     next_key: null
     total: "0"
   ```

   akt的精度是6，所以余额应该是 100000000 uakt = 100 akt

#### Deploy Ghost

1. Configure the `deploy.yml` file according to the application [deployment tutorial](https://docs.akash.network/v/master/guides/deploy)

   reate & edit `deploy.yml`  file

   ```
   touch deploy.yml
   vi deploy.yml
   ```

   add the following

   ```
   ---
   version: "2.0"
   
   services:
     ghost:
       image: ghost
       expose:
         - port: 2368
           as: 80
           to:
             - global: true
   profiles:
     compute:
       ghost:
         resources:
           cpu:
             units: 1
           memory:
             size: 1Gi
           storage:
             size: 5Gi
     placement:
       westcoast:
         attributes:
           organization: ovrclk.com
         signedBy:
           anyOf:
             - "akash1vz375dkt0c60annyp6mkzeejfq0qpyevhseu05"
         pricing:
           ghost: 
             denom: uakt
             amount: 2000
   
   deployment:
     ghost:
       westcoast:
         profile: ghost
         count: 1
   ```
   
2. Deploy

   ```
   akash tx deployment create deploy.yml --from $KEY_NAME --node $AKASH_NODE --chain-id $AKASH_CHAIN_ID -y
   ```

   the output is as follows

   ```
   {
     "height": "183906",
     "txhash": "FC58F52F49E91808A4751E5A5603C34489B253F59E40F7058E1CC5E70F036CA1",
     "codespace": "",
     "code": 0,
     "data": "0A130A116372656174652D6465706C6F796D656E74",
     "raw_log": "[{\"events\":[{\"type\":\"akash.v1\",\"attributes\":[{\"key\":\"module\",\"value\":\"deployment\"},{\"key\":\"action\",\"value\":\"deployment-created\"},{\"key\":\"version\",\"value\":\"f2b5e760ca93c53548737d76494db89b08d811f7e9b533aa07fd934bf51d6707\"},{\"key\":\"owner\",\"value\":\"akash1j8s87w3fctz7nlcqtkl5clnc805r240443eksx\"},{\"key\":\"dseq\",\"value\":\"19553\"}]},{\"type\":\"message\",\"attributes\":[{\"key\":\"action\",\"value\":\"create-deployment\"}]}]}]",
     "logs": [
       {
         "msg_index": 0,
         "log": "",
         "events": [
           {
             "type": "akash.v1",
             "attributes": [
               {
                 "key": "module",
                 "value": "deployment"
               },
               {
                 "key": "action",
                 "value": "deployment-created"
               },
               {
                 "key": "version",
                 "value": "f2b5e760ca93c53548737d76494db89b08d811f7e9b533aa07fd934bf51d6707"
               },
               {
                 "key": "owner",
                 "value": "akash1j8s87w3fctz7nlcqtkl5clnc805r240443eksx"
               },
               {
                 "key": "dseq",
                 "value": "19553"
               }
             ]
           },
           {
             "type": "message",
             "attributes": [
               {
                 "key": "action",
                 "value": "create-deployment"
               }
             ]
           }
         ]
       }
     ],
     "info": "",
     "gas_wanted": "200000",
     "gas_used": "53806",
     "tx": null,
     "timestamp": ""
   }
   ```

3. View the status of just deployed

   ```
   akash query market lease list --owner $ACCOUNT_ADDRESS --node $AKASH_NODE --state active
   ```

   the output is as follows

   ```
   - lease_id:
       dseq: "183480"
       gseq: 1
       oseq: 1
       owner: akash15rrya4ayu8yewh3d2agg552phtttpkghpd0paf
       provider: akash174hxdpuxsuys9qkauaf57ym5j8dm4secnz6jd7
     price:
       amount: "331"
       denom: uakt
     state: active
   pagination:
     next_key: null
     total: "0"
   ```

   record several variables of the previous output for use

   ```
   export PROVIDER=akash174hxdpuxsuys9qkauaf57ym5j8dm4secnz6jd7
   export DSEQ=183480
   export OSEQ=1
   export GSEQ=1
   ```

4. Upload `Manifest`

   ```
   akash provider send-manifest deploy.yml --node $AKASH_NODE --dseq $DSEQ --oseq $OSEQ --gseq $GSEQ --owner $ACCOUNT_ADDRESS --provider $PROVIDER
   ```

5. View the specific information of the deployment

   ```
   akash provider lease-status --node $AKASH_NODE --dseq $DSEQ --oseq $OSEQ --gseq $GSEQ --provider $PROVIDER --owner $ACCOUNT_ADDRESS
   ```

   the output is as follows

   ```
   {
       "services": {
           "ghost": {
               "name": "ghost",
               "available": 1,
               "total": 1,
               "uris": [
                   "538dqfcf7tdjtbp250vlepvi5k.provider4.akashdev.net"
               ],
               "observed-generation": 0,
               "replicas": 0,
               "updated-replicas": 0,
               "ready-replicas": 0,
               "available-replicas": 0
           }
       },
       "forwarded-ports": {}
   }
   ```
   
6. Access the deployed application

   `http://538dqfcf7tdjtbp250vlepvi5k.provider4.akashdev.net/`

7. Close deployment

   ```
   akash tx deployment close --node $AKASH_NODE --chain-id $AKASH_CHAIN_ID --dseq $DSEQ --owner $ACCOUNT_ADDRESS --from $KEY_NAME -y
   ```



​	The deployment of the tutorial is now complete.

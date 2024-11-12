
# Movement Follower Node



## Getting Started


To do this, ensure you have `nix` installed. We recommend the Determinate Systems `nix` installation script. You can find it [here](https://determinate.systems/posts/determinate-nix-installer/).

```bash
curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install
```

After installing `nix`, clone the Movement repository and open the `nix-shell` environment.

```bash
git clone https://github.com/movementlabsxyz/movement
cd movement
nix develop
```

This should install all dependencies needed to work on the Movement Follower Node.

You can now either run the follower node natively or with our containers via the provided `just` commands.

First create, an environment file for the follower node. The example below is for the Movement Testnet. Comments are made on how to change the environment file for other networks.

```bash
CONTAINER_REV=<latest-commit-hash>
DOT_MOVEMENT_PATH=./.movement
MAPTOS_CHAIN_ID=250 # change this to the chain id of the network you are running
MOVEMENT_SYNC="follower::mtnet-l-sync-bucket-sync<=>{maptos,maptos-storage,suzuka-da-db}/**" # change to the sync bucket for the network you are running
M1_DA_LIGHT_NODE_CONNECTION_PROTOCOL=https
M1_DA_LIGHT_NODE_CONNECTION_HOSTNAME="m1-da-light-node.testnet.bardock.movementlabs.xyz" # changes this to the hostname of the m1_da_light_node_service on network you are running
M1_DA_LIGHT_NODE_CONNECTION_PORT=443
# you may need to provide AWS credentials for the Amazon SDK to properly interact with the sync bucket
# often this will be picked up appropriately if your environment is configured to use AWS
# the bucket has public read access so you may not need to provide credentials
AWS_ACCESS_KEY_ID=<your-access-key>
AWS_SECRET_ACCESS_KEY=<your-secret-access-key>
AWS_DEFAULT_REGION=us-west-1 # change this to match the region of the sync bucket
AWS_REGION=us-west-1 # change this to match the region of the sync bucket
```

Example for APTOS bardock testnet:
```bash
CONTAINER_REV=102389ecfc4de1f7ec7da9902bb1758afaa2c9f6
DOT_MOVEMENT_PATH=./.movement
MAPTOS_CHAIN_ID=250 # change this to the chain id of the network you are running
MOVEMENT_SYNC="follower::mtnet-l-sync-bucket-sync<=>{maptos,maptos-storage,suzuka-da-db}/**" 
M1_DA_LIGHT_NODE_CONNECTION_PROTOCOL=https
M1_DA_LIGHT_NODE_CONNECTION_HOSTNAME="m1-da-light-node.testnet.bardock.movementlabs.xyz" 
M1_DA_LIGHT_NODE_CONNECTION_PORT=443
# you may need to provide AWS credentials for the Amazon SDK to properly interact with the sync bucket (CREATE from 
AWS_ACCESS_KEY_ID=xxxxxxxxxxx 
AWS_SECRET_ACCESS_KEY=xxxxxxxx
AWS_DEFAULT_REGION=us-west-1
AWS_REGION=us-west-1
```

To run natively you can use the following command:

```bash
source .env
just suzuka-full-node native build.setup.follower -t=false
```

To run with containers you can use the following command:

```bash
just suzuka-full-node docker-compose follower
```

To check on the status of the service under either runner, run:

```bash
curl http://localhost:30731/v1
```

You should see a `ledger_version` field CLOSE to the other values on the network, e.g., [https://aptos.testnet.bardock.movementlabs.xyz/v1](https://aptos.testnet.bardock.movementlabs.xyz/v1).

## Deployment and Advanced Usage
For deployment and advanced usage, we recommend you use our [provided Ansible scripts](../../ansible/follower-node/README.md).


  

1. Get the container version
```bash
# From /home/${USER}
git clone https://github.com/movementlabsxyz/movement.git
cd movement
CONTAINER_REV=$(git rev-parse HEAD)
echo "CONTAINER_REV=${CONTAINER_REV}"
```


2. Run the playbook
Make sure you can connect to the host where you want test or like in this case, a test
ec2 instance
```bash
ssh ubuntu@ec2-54-215-191-59.us-west-1.compute.amazonaws.com
```

```bash
ansible-playbook --inventory ec2-54-215-191-59.us-west-1.compute.amazonaws.com, \
                 --user ubuntu  \
                 --extra-vars "movement_container_version=${CONTAINER_REV}" \
                 --extra-vars "user=ubuntu" \
                 docs/movement-node/run/ansible/suzuka-full-node.yml
```

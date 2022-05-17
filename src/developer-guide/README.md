# Developer Guide
This guide is aimed at people who want to develop on Anoma itself, or write custom transactions or validity predicates. It assumes you can already [build Anoma from source](../user-guide/install.md).

## Setting up a local chain
Every Anoma network has a unique chain ID e.g. `dev.a9b21fcde663a587810c33bbbe`, and lives in a directory like `.anoma/dev.a9b21fcde663a587810c33bbbe/`. This chain directory is initialized when `anomac utils join-network` is run to join the chain.

For development, you can initialize a new network from scratch, where you run the sole validator node on your local machine. It can then be interacted with similarly to a real devnet or testnet node.

Decide on the prefix for your local network's chain ID upfront. A prefix like `dev` could result in a final chain ID like `dev.a9b21fcde663a587810c33bbbe`.

```bash
export ANOMA_CHAIN_PREFIX='dev'
```

### Generating your genesis validator keys
Follow the [genesis validator setup guide](../user-guide/genesis-validator-setup.md) to generate all necessary validator keys and configuration. Make sure the validator alias is exported in your shell for further commands.

```bash
export ALIAS='validator-dev'
```

### Creating the network configuration file
To create the local chain, you need a network configuration file. The below example has:
- a validator with alias 'validator-dev'
- a single predefined fixed supply token (XAN)
- a faucet with a supply of XAN
- some other parameters

Replae the example validator configuration in the TOML below (i.e. the bit in between `### ↓ from pre-genesis` and `### ↑ from pre-genesis`) with the genesis validator configuration generated in the previous step, which should look similar.

```toml
# the genesis_time can be any time in the past - since we will want to start this chain immediately
genesis_time = "2022-01-01T00:00:00.00Z"

### ↓ from pre-genesis
[validator.validator-dev]
consensus_public_key = "00c058c248c2748b9d0a24a163f798cde6058e9dfcc8cf3886cf19ee747ca5c8d2"
account_public_key = "00be44cb3d66d4271c4f4f4afa2638944aa66e78cea6eebbc9a9768bf73e6cffcc"
staking_reward_public_key = "007dec4597671916d4cecb2df07023ec51f3aee1fce1becb840ea8f8be77ef5fe6"
protocol_public_key = "003ff2440a2bb247c7be9fa35155e7607a56170fde48318173c5000b22a90d7e68"
dkg_public_key = "60000000497e31d46e6867e07ce881dec55c909efb307d1b081464461054c6f28c8886c82abb637e367d19592b6c6e37dd85600225975c4b62f80b36acfcaba654ef00e8c6e1db713e966ba39da528d8f093d3d552f4767ee2840471d3bff1485b033294"
net_address = "127.0.0.1:26656"
tendermint_node_key = "0005971884fc4a1f678d0550074f073083aee2dd5d5f935e7b2fb1c2fb3131fb1f"
### ↑ from pre-genesis
tokens = 6714000000
non_staked_balance = 1000000000000
validator_vp = "vp_user"
staking_reward_vp = "vp_user"

[token.XAN]
address = "atest1v4ehgw36x3prswzxggunzv6pxqmnvdj9xvcyzvpsggeyvs3cg9qnywf589qnwvfsg5erg3fkl09rg5"
vp = "vp_token"

[token.XAN.balances]
atest1v4ehgw36gc6yxvpjxccyzvphxycrxw2xxsuyydesxgcnjs3cg9znwv3cxgmnj32yxy6rssf5tcqjm3 = 9223372036854

[established.faucet]
address = "atest1v4ehgw36gc6yxvpjxccyzvphxycrxw2xxsuyydesxgcnjs3cg9znwv3cxgmnj32yxy6rssf5tcqjm3"
vp = "vp_testnet_faucet"

# Wasm VP definitions

# Default user VP
[wasm.vp_user]
filename = "vp_user.wasm"
sha256 = "8bb90e43d53156c26e60082693fcb35fa0eb4a1ca3adaa5fb9e8b2ec3477ff95"

# Token VP
[wasm.vp_token]
filename = "vp_token.wasm"
sha256 = "7c9bb99a9eb45bdb62cda38b2d09b45eda22138910db5b194aa8c1ba1fdfda78"

# Faucet VP
[wasm.vp_testnet_faucet]
filename = "vp_testnet_faucet.wasm"
sha256 = "cdc7d2c6118c3f4909c700a63b97f9c734c463ac32cb30164737a2d85d6ee873"


# General protocol parameters.
[parameters]
# Minimum number of blocks in an epoch.
min_num_of_blocks = 2
# Minimum duration of an epoch (in seconds).
min_duration = 10
# Maximum expected time per block (in seconds).
max_expected_time_per_block = 30

# Proof of stake parameters.
[pos_params]
# Maximum number of active validators.
max_validator_slots = 4
# Pipeline length (in epochs). Any change in the validator set made in
# epoch 'n' will become active in epoch 'n + pipeline_len'.
pipeline_len = 2
# Unbonding length (in epochs). Validators may have their stake slashed
# for a fault in epoch 'n' up through epoch 'n + unbonding_len'.
unbonding_len = 4
# Votes per token (in basis points, i.e., per 10,000 tokens)
votes_per_token = 10
# Reward for proposing a block.
block_proposer_reward = 100
# Reward for voting on a block.
block_vote_reward = 1
# Portion of a validator's stake that should be slashed on a duplicate
# vote (in basis points, i.e., 500 = 5%).
duplicate_vote_slash_rate = 500
# Portion of a validator's stake that should be slashed on a light
# client attack (in basis points, i.e., 500 = 5%).
light_client_attack_slash_rate = 500

# Governance parameters.
[gov_params]
# minimum amount of xan token to lock
min_proposal_fund = 500
# proposal code size in bytes
max_proposal_code_size = 300000
# proposal period length in epoch
min_proposal_period = 3
# maximum number of characters in the proposal content
max_proposal_content_size = 5000
# minimum epochs between end and grace epoch
min_proposal_grace_epochs = 6

[treasury_params]
max_proposal_fund_transfer = 10000
```

Save this file somewhere (e.g. `dev-network-configuration.toml`).

### Creating the network release

```bash
export ANOMA_NETWORK_CONFIG_PATH='dev-network-configuration.toml'

cargo run --bin anomac -- utils init-network \
	--chain-prefix $ANOMA_CHAIN_PREFIX \
	--genesis-path $ANOMA_NETWORK_CONFIG_PATH \
	--wasm-checksums-path wasm/checksums.json \
	--unsafe-dont-encrypt
```

This will output something like:
```
Generating established account faucet key...
Warning: The keypair will NOT be encrypted.
Derived chain ID: dev.a9b21fcde663a587810c33bbbe
Genesis file generated at .anoma/dev.a9b21fcde663a587810c33bbbe.toml
Release archive created at /opt/workspace/heliax/repos/anoma/primary/dev.a9b21fcde663a587810c33bbbe.tar.gz
```

Put the chain ID from the output in your shell.

```shell
export CHAIN_ID='dev.a9b21fcde663a587810c33bbbe'
```

Also, remove the existing chain directory that was generated when running `init-network` e.g. `.anoma/abcipp.15529e525fab1c51f36d7c3/`, as this will interfere with joining the network in the next step.


### Joining your network

In order for `anomac utils join-network` to be able to join the chain, you need to "fake" a release server. This can most easily be done by running `python3 -m http.server` in the same directory, then:

```bash
export ANOMA_NETWORK_CONFIGS_SERVER='http://localhost:8000'

cargo run --bin anomac -- utils join-network \
	--genesis-validator $ALIAS \
	--chain-id $CHAIN_ID
```

`.anoma/global-config.toml` should now show your new chain as the chain ID e.g.

```
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: .anoma/global-config.toml
       │ Size: 52 B
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ default_chain_id = "abcipp.15529e525fab1c51f36d7c3"
───────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

You should update this value if you ever want to switch between local chains.

### Starting your ledger node

```shell
export ANOMA_LOG='debug'
export ANOMA_TOKIO_THREADS=2
export ANOMA_RAYON_THREADS=2
cargo run --bin anoman ledger
```

At this point you should be able to follow the [user-guide](../user-guide/README.md) using your local chain instead of a testnet.

## Writing a custom transaction
TODO - How to write a simple tx, compile the WASM blob and then submit it to your local chain using `anomac`.

## Writing a custom validity predicate
TODO - How to write a simple vp, compile the WASM blob and then initialize an account with it on the local chain. Then submit a tx to prove it works.

## Running tests
TODO
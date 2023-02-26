# Foundry Deploying to Goerli TestNet

Accompanies myyoutube video of the same name

If you are new to Foundry, I would recommend you to go through the [Foundry Book](https://book.getfoundry.sh/).

Also this cheat sheet is very helpful - [Foundry Cheat Sheet](https://github.com/dabit3/foundry-cheatsheet).

## Prerequisites

- [Rust](https://www.rust-lang.org/tools/install)
- [Install Foundry](https://book.getfoundry.sh/getting-started/installation)

## Useful Commands

### Build the contract

```bash
forge build
```

### Test the contract

```bash
forge test
```

### See the logs and tracebacks while testing

```bash
forge test -vvvv
```

### Gas report

```bash
forge test --gas-report
```

### Deploying the contract to anvil

```bash
forge create src/CrowdFunding.sol:CrowdFunding --private-key=<private-key-from-anvil>
```

### Deploying the contract to anvil using the deployment script

You need to have a env file with PRIVATE_KEY of any of the account in anvil for this to work.

```bash
cp .env.example .env
```

```bash
forge script script/CrowdFunding.s.sol --fork-url http://localhost:8545 --broadcast
```

## Interacting with the contract deployed on anvil

After deploying the contract get the contract address from the output

```bash
export CONTRACT_ADR=<contract-address>
```

Setting some variables for ease of use

```bash
export CAMPAIGN_OWNER_PRIV_KEY=<private-key-of-first-account-in-anvil>

export CAMPAIGN_OWNER_ADDRESS=<address-of-first-account-in-anvil>

export CAMPAIGN_DONOR_PRIV_KEY=<private-key-of-first-account-in-anvil>
```

### Creating a campaign

```bash
cast send $CONTRACT_ADR "createCampaign(address,string,string,uint256,uint256,string)"  $CAMPAIGN_OWNER_ADDRESS "Campaign 1" "Campaign 1 Description" 10ether <current-epoch-time + 300> "https://i.kym-cdn.com/photos/images/newsfeed/002/205/307/1f7.jpg" --private-key=$CAMPAIGN_OWNER_PRIV_KEY
```

**_NOTE:_** For the time being, I couldn't find any other way for setting the deadline, what I did is got the current Unix epoch time from here - https://www.epochconverter.com/ and added it to the command: 5\*60 = 300 seconds to it.

### Get the campaign details

```bash
cast call $CONTRACT_ADR "getCampaign(uint256)(address,string,string,uint256,uint256,uint256,string)" 0
```

### Contribute to the campaign

```bash
cast send $CONTRACT_ADR "donateToCampaign(uint256)" 0 --value 10ether --private-key=$CAMPAIGN_DONOR_PRIV_KEY
```

### Withdraw the funds

```bash
cast send $CONTRACT_ADR "withdraw(uint256)" 0 --private-key=$CAMPAIGN_OWNER_PRIV_KEY
```

### To see the balance of the campaign owner

```bash
cast balance $CAMPAIGN_OWNER_ADDRESS
```

## Deploying to Goerli

Adding in some variables

```bash
export GOERLI_URL=<goerli-url-from-infura-or-alchemy>

export PRIVATE_KEY=<private-key-from-metamask>

export ETHERSCAN_KEY=<etherscan-api-key>
```

### Deploying the contract to Goerli along with the verification

```bash
forge create --rpc-url $GOERLI_URL --private-key $PRIVATE_KEY src/CrowdFunding.sol:CrowdFunding --etherscan-api-key $ETHERSCAN_KEY --verify
```

### Interacting with the contract deployed on Goerli

You can use the same commands as above, but you need to change the contract address to the one you got from the output of the above command and add `--rpc-url $GOERLI_URL` to the commands.

Example for getting the campaign details

```bash
cast call $CONTRACT_ADR "getCampaign(uint256)(address,string,string,uint256,uint256,uint256,string)" 0 --rpc-url $GOERLI_URL
```
# Launching our own coin
In the previous quest we looked at creating a uniswap based currency exchanger. If you have not done that already, you should ().

In this Quest, instead of using DAI as our crypto currency, we’ll deploy our own crypto currency, that not only will we be able to use, but anybody can start buying too! At the end of this quest, we’ll put up our coin on an exchange. Who knows, your coin might moon!
## ERC20
Quick recap - in the last quest we saw that every crypto currency on Ethereum is a smart contract. This smart contract has a few functions that allow us to do the standard things that you would with any currency. The community has come up with a standard for this. So every contract has the same set of functions. Every wallet like metamask is designed expecting those functions to exist. This standard was first proposed by Vitalik and team here : [https://github.com/ethereum/EIPs/blob/f98a60e1b1590c9cfa0bfe66c4e6a674e54cd386/EIPS/eip-20.md](https://github.com/ethereum/EIPs/blob/f98a60e1b1590c9cfa0bfe66c4e6a674e54cd386/EIPS/eip-20.md)

It is a simple proposal saying any smart contract written to launch a new crypto currency (aka token), must implement the mentioned functions. The functions have been explained in the above EIP. 

An EIP is an Ethereum Improvement Proposal. Once approved by the community, it is converted into an ERC (Ethereum request for comment). The EIP20 was accepted by the community. Now every developer uses this standard to launch a crypto currency. Every other software in the ecosystem including wallets (like metamask) and exchanges (like uniswap) use this standard to interact with the smart contracts - because they know what to expect.

Remember we defined a generic erc20 Interface and used the same interface for any token address that was sent? That was possible only because every erc20 token implements the same set of functions (interface).

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/bd2e9de4-c9dc-4a1d-8c12-358ea1d8f666.jpg)

Now, on to building ...

## Writing a new contract 

First up please visit the ERC 20 specification here  

[https://github.com/ethereum/EIPs/blob/f98a60e1b1590c9cfa0bfe66c4e6a674e54cd386/EIPS/eip-20.md](https://github.com/ethereum/EIPs/blob/f98a60e1b1590c9cfa0bfe66c4e6a674e54cd386/EIPS/eip-20.md)

You will have a clear understanding of what functions we need to create. Let us first create a simple coin contract with all the mentioned functions. You can copy and paste the function signatures from the EIP. We’ll fill them up through this Quest.

[https://remix.ethereum.org/\#version=soljson-v0.8.4\+commit.c7e474f2.js&optimize=false&runs=200&gist=0727bbef01893ab4569f53fb274c90be&evmVersion=null](https://remix.ethereum.org/#version=soljson-v0.8.4+commit.c7e474f2.js&optimize=false&runs=200&gist=0727bbef01893ab4569f53fb274c90be&evmVersion=null)

Ready made contract, if you’re too lazy to copy paste the signatures.

Let’s start filling up the basics.

The name() function should return the name of your token. It could be any name whatsoever.

The symbol() should return a, well, symbol for your token. This symbol is usually a 3 or 4 character short hand for your token. Eg. ETH (for ethers), COMP (for compound), UNI (for uniswap). This is the symbol a lot of exchanges use in their tickers for simplicity. 

Check out list of all the present tokens here [https://coinmarketcap.com/](https://coinmarketcap.com/)

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/519ee57a-d133-47f7-9f2c-04b2759b9409.jpg)

# 

Note that we have modified the signature a little bit. Instead of returns(string), we have changed it to returns(string memory). The memory keyword tells Solidity that after returning the variable, it can erase it from the memory stack.

There are two such modifiers. Memory and Storage. 

Memory is for impersistent storage. That is after the function call the variable can be killed and deleted.

However a Storage variable is stored persistently. The data will be accessible across function calls.

In our case we want the function name() to return the name string. This function doesn’t modify any variable that we’d need to store for use by other functions.
## Defining the properties of this currency
There are two properties we need to define for the currency itself. 

The first is the total supply. This defines how many coins of this currency will ever be in circulation.

There are two types of currencies that are popular right now : Inflationary and Fixed.

Inflation is one in which the number of coins in circulation keeps increasing with time. I.e the function totalSupply() returns a higher number each time you call it. ETH is an inflationary crypto currency.

On the other hand, fixed supply currencies define how many coins will ever be in circulation. This creates scarcity, and drives up the prices on the exchanges when demand for the coins increases. Bitcoin is a fixed supply crypto currency, albeit not an ERC20 token. COMP (compound) is a fixed supply ERC20 token that has only 10M coins that will ever be in circulation.

We’ll start with a fixed supply token. Let us start with 10M coins.

Let us define the second parameter of our currency that is decimals().

Solidity doesn’t have support for decimals (double and float) yet. But it does support big integers. That is also why Ethers are always measured in Wei. So instead of saying 0.000000000000000001 ETH, we say 1 Wei. 

The decimals() tells the world that every balance reported by this contract must be divided by ten to the power of this to get the actual value. So if we set our decimal to be 8, and a user has a balance of 1000, it really means they have a balance of 0.00001 coins.

How divisible do you want these coins to get? Really, an open question. But for now let us set this at 8. 

[https://remix.ethereum.org/\#version=soljson-v0.8.4\+commit.c7e474f2.js&optimize=false&runs=200&gist=58630975c136bad48caa70d11c8a7410&evmVersion=null](https://remix.ethereum.org/#version=soljson-v0.8.4+commit.c7e474f2.js&optimize=false&runs=200&gist=58630975c136bad48caa70d11c8a7410&evmVersion=null)
## Dealing with money
Now that we have all the logistics in place let’s start dealing with real money.

The first function to look at is balanceOf().

Given a user address, we must return the balance of that person. 

In order to enable this functionality, we’ll define a mapping so that we can store the balances.

mapping(address => uint)

[https://remix.ethereum.org/\#version=soljson-v0.8.4\+commit.c7e474f2.js&optimize=false&runs=200&gist=f701d2de96a870337ea6a77876b9497f](https://remix.ethereum.org/#version=soljson-v0.8.4+commit.c7e474f2.js&optimize=false&runs=200&gist=f701d2de96a870337ea6a77876b9497f)

Similarly, when transfer is called we will deduct the balance from one user to another.

You’ll notice the function has two parameters : to and value (aka amount). But from whom?

Try writing the transfer function before you check out the answer :

[https://remix.ethereum.org/\#version=soljson-v0.8.4\+commit.c7e474f2.js&optimize=false&runs=200&gist=65489c972c2911a0765b4c5e78b308ce&evmVersion=null](https://remix.ethereum.org/#version=soljson-v0.8.4+commit.c7e474f2.js&optimize=false&runs=200&gist=65489c972c2911a0765b4c5e78b308ce&evmVersion=null)

We’ve added an assert statement so that we transfer money only if the sender actually has that much money. If the assert fails the function will terminate.
## Minting a million coins for yourself
Since you are the person who is creating this contract and currency, we can set some rules favourable to us. 

We will give ourselves a balance of 1M coins!

For this we’ll use a constructor to the smart contract. 

The constructor is called when the contract is deployed. Just like every other function, the msg object is populated and the msg.sender is set to the address of the user deploying the contract. 

The initial million tokens can be set when the constructor is called. The constructor will be called exactly once in the lifetime of the contract. It is also a good idea to store the deployer’s address as a variable in the contract, so that if you want to give some special permissions to this user, you can do that using this variable. 

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/5ef9786d-6afe-4465-983e-2dc117225e6c.jpg)

We’ve just pulled out 1M coins from thin air! 

So earlier we mentioned that the total supply is 10M coins. We’ve minted(taken up) 1M coins into our own address. How will the next 9M coins get into circulation?

You can use whatever logic you want but we’re going to use a very simple logic. Every 10th block on Ethereum, 10 coins will be rewarded into the wallet of a person on a first come first serve basis.

A new block is created on ethereum about once every 20 seconds.

So every 200 seconds one lucky person will be able to see that no one has claimed the reward for a certain 10th block & claim it.

The process for claiming these rewards are

- Check if the current block number is a multiple of 10
- Check if the reward has already been given to someone
- If the reward has not yet been given out, claim it!

We’ll call this process of digging for rewards as mining.

Let’s add a function called mine(). Can you populate this function?

Let us also add two more functions to return the current block number (getBlockNumber() )and whether a certain block has been mined (isMined(blockNumber)).

[https://remix.ethereum.org/\#version=soljson-v0.8.4\+commit.c7e474f2.js&optimize=false&runs=200&gist=9f38e3fe3d29e82c8dd81bb8b1b3ecf5](https://remix.ethereum.org/#version=soljson-v0.8.4+commit.c7e474f2.js&optimize=false&runs=200&gist=9f38e3fe3d29e82c8dd81bb8b1b3ecf5)
## Token approvals
These currencies allow for what is called approvals. Think of it as a parent letting their teenage kids spend money. ERC20 specifications require us to implement a function in which a user can give approval to another account to spend a designated number of coins.

This functionality is particularly useful when a user wants to allow a contract to spend money on their behalf. Just like what we saw in the previous quest. We wanted the contract to take our DAI and send it to Uniswap on our behalf. So what we did was, gave approval to the contract to use a certain number of DAI from our account.

The first function is the approve() function. Just like the balances, we must maintain what is the allowance for each user. Each user can give approvals to multiple other users. So, our data type will be

mapping(address => mapping(address => uint)) allowances

Allowances\[user1\]\[user2\] = 10; implies user1 has approved user2 to spend 10 coins.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/8e8095a2-9ec7-4d59-aed2-13ef74e22dde.jpg)

Lastly, we’ll populate transferFrom.

A user can transfer from another user’s account if they have the allowance to do so. 

When this function is called, we need to check

1. If the source user actually has those many coins in balance
2. The spender (user calling this function) has allowance to spend the designated amount

Can you write the function with these two checks and execute the following?

1. Debit amount from the “from user”
2. Credit to the “to user”
3. Reduce the allowance of the spender

[https://remix.ethereum.org/\#version=soljson-v0.8.4\+commit.c7e474f2.js&optimize=false&runs=200&gist=e32a9f4597d8afc740592156c1ba3859&evmVersion=null](https://remix.ethereum.org/#version=soljson-v0.8.4+commit.c7e474f2.js&optimize=false&runs=200&gist=e32a9f4597d8afc740592156c1ba3859&evmVersion=null)
## Adding events
There’s another requirement that the ERC20 specification mandates. 

It requires that events be triggered for allowance and transfers.

Any program like a web serverver, web app or iPhone application.

The events are mentioned here. You can copy paste the signatures to define the event.

[https://github.com/ethereum/EIPs/blob/f98a60e1b1590c9cfa0bfe66c4e6a674e54cd386/EIPS/eip-20.md\#events](https://github.com/ethereum/EIPs/blob/f98a60e1b1590c9cfa0bfe66c4e6a674e54cd386/EIPS/eip-20.md#events)

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/edf79015-6f7f-4ad3-a989-221823830622.jpg)

Now that the events have been defined, they can be emitted. When an event is emitted, the other applications will be able to listen to them.

Transfer has to be emitted whenever a transfer is made. I.e. whenever the transfer() and transferFrom() are successful - so that, say the iphone app, can update the new balance of the user after the transfer in real time.

Approve must be emitted when the approval is made, so that the application can notify the spender that they now have an allowance waiting to be spent!

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/c01fab58-d88f-42d1-825e-5c3e1fb3c356.jpg)

Can you add the appropriate event emissions to transfer(), transferFrom() and approve() ?

[https://remix.ethereum.org/\#version=soljson-v0.8.4\+commit.c7e474f2.js&optimize=false&runs=200&gist=a577a320809b2b21b6665c1afe7de821](https://remix.ethereum.org/#version=soljson-v0.8.4+commit.c7e474f2.js&optimize=false&runs=200&gist=a577a320809b2b21b6665c1afe7de821)
## Adding to metamask
Now that we have written the entire contract to the specifications of ERC20, let’s deploy it to Ropsten.

Once it is deployed to ropsten, you can add this to your metamask wallet!

Copy the address of the deployed contract.

Open metamask

Go to the assets tab

Tap on Add Token.

Paste the contract address

Token symbol and decimal should automatically get filled up - suggesting that our coin has been recognized by metamask because it adheres to the erc20 specs.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/b48a53b2-de1d-4ea8-aa49-45afd54f4630.jpg)

Once you add the tokens you will see that you have a million tokens!

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/791e5548-d111-453b-88a7-425e04a29455.jpg)

Congratulations on launching your coin on Ropsten! You can send and receive tokens just like any other token!
## Listing on Uniswap
What fun if people can’t buy these coins and speculate?

Let us list our new token on Uniswap - the place where all the traders are continuously looking for new coins to buy!

First we need to create a pool by what is called adding liquidity. Adding liquidity is like listing the tokens for sale on Uniswap.

Head to app.uniswap.org 

Go to pool

Select v2 liquidity from the dropdown “more”

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/bd6df9d3-eec8-4569-a59e-ccd85b3f5600.jpg)

Then select create a pair

We will create a pair with ETH. So that people can pay in ETH and retrieve our tokens (M20). 

Let’s set the initial price of 10,000 tokens to be 1 ETH

Enter 1 eth against ETH in the first row

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/9ef903e1-8bf2-42a3-ab3a-a69fb3608d96.jpg)

Then tap the select a token.

In the top input box paste the address of our contract that you deployed earlier.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/c4369af0-f267-489e-8513-2a064cd36cd9.jpg)

You’ll notice that Uniswap automatically detects the name and symbol of this coin. Tap import.

Enter M20 to be 10000.

By doing so we’re setting the initial price. 10000 M20 coins now cost 1ETH. This is the initial price. Uniswap will alter the price based on some rules as and when there is high or low demand for these coins.

Then approve M20. By approving you’ll invoke the approve function that you had written in your contract. But wait.

Before you do that, go back to remix - let us also listen to the events. Check listen on network in your console.

Then go back to Uniswap and tap approve!

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/b5c50b75-ebf4-44c2-b09a-27f7d388bf01.jpg)

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/9c944b32-42c7-4d99-905d-d1be08350ab3.jpg)

Meta mask will open up and ask you to confirm the transaction.

Once this is done, 

Go to add liquidity

And enter the same values

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/987b6a6c-4c2d-4e41-b770-689c7e1e9a7a.jpg)

Tap supply

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/ce4dc0ac-c587-4ed8-b59e-d4478cbd3419.jpg)

This time metamask will popup asking to transfer 1ETH. 

To know more about uniswap liquidity pools check this blog - [https://docs.uniswap.org/protocol/V2/concepts/core-concepts/pools](https://docs.uniswap.org/protocol/V2/concepts/core-concepts/pools)

Once the transaction is complete, you’ll see the number of coins in your metamask wallet has gone down. That means it is now ready for trading!

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/06184a34-5755-4ecd-b027-3b3f9669c3ce.jpg)

Then go to the swap tap

You will be able to buy M20 coins back by paying an amount in eth!

So anyone can buy the coins from uniswap. When more people buy, the price goes up. You still have 990K coins with you, they’re currently worth 99ETH. Now you just have to go find the traders! 

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/101dab90-8268-4986-86f5-1e410877a404.jpg)
## Activity
Create a new coin in which you, the deployer of the contract, earn each time there is a transfer made - as a commission! :)

Try depositing your new coin into the smart bank account we had written in the previous quest 

While mining, in the mine() function, make sure the number of coins minted is never more than total supply.
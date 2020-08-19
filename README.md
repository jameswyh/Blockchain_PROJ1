# Open_Auction_Dapp
Author: Yihui Wang
## Description
The goal of this project is to develop a smart contract for open auction. The general idea of the auction contract is that everyone can send their bids during a bidding period. The bids already include sending money / ether in order to bind the bidders to their bid. If the highest bid is raised, the previously highest bidder gets their money back. After the end of the bidding period, the contract has to be called manually for the beneficiary to receive their money - contracts cannot activate themselves.
## Requirements
1. NodeJS v8.9.4 or later
2. Windows, Linux or Mac OS X

### Install Truffle 
Truffle is made for building dApps using the Ethereum Virtual Machine (EVM) by providing a development environment, testing framework, and asset pipeline. To install, run the following command:`npm install −g truffle`<br/>
On a Unix-based system, you may need to add "sudo" before the command. You can verify the installation by running `truffle version` on a terminal, you should see an output similar to the following:<br/>
```
Truffle v5.1.12 (core: 5.1.12)
Solidity v0.5.16 (solc−js)
Node v8.10.0
Web3.js v1.2.1
```

### Install Ganache 
Ganache is a personal blockchain that allows developers tocreate smart contracts, dApps, and test software that is available as a desktop application and commandline tool for Windows, Mac, and Linux. To install, follow the instructions on https://www.trufflesuite.com/docs/ganache/quickstart. Note: please choose version 2.1.2, do not choose any beta versions.

### Deploy the contract on Ganache
Start the Ganache software and run the following command in the root directory: `truffle migrate` <br/>
By default, Ganache provides 10 Ethereum accounts, each has 100 ETH, to interact with the smart contract. The first account is the one deploying the contract and becomes the beneficiary.<br/>
You can interact with your contract using Truffle console. From the root direc- tory, run:`truffle console`<br/>

### Function Description:
```
1    function bid() public payable {
2        require(msg.value > highestBid);
3        if(highestBidder != address(0x0)){
4          pendingReturns[highestBidder] = highestBid;
5        }
6        highestBid = msg.value;
7        highestBidder = msg.sender;
8    }
```
In the line 2, I use a “require” to ensure that the bid is higher than the current highest bid, or the bid will be invalid. From line 3 to 4, I store the previous bid in pendingReturns, which can be withdrawn when the bidder triggers withdraw() function. In line 6 to 7, the newest highest bid the address of the bidder is updated.

```
1    function withdraw() public returns (bool) {
2        uint amount = pendingReturns[msg.sender];
3        if(amount > 0){
4          pendingReturns[msg.sender] = 0;
5        }
6        if(!msg.sender.send(amount)){
7          pendingReturns[msg.sender] = amount;
8          return false;
9        }
10        return true;
11    }
```
In the line 2 to 5, I store the bid of the bidder in a variable “amount”, and then check whether the amount is more than 1. If the amount is more than 1, I set the bid in the pendingReturns to 0, which can avoid the reentrancy attack discussed in class. From line 6 to 8, if the bid does not send back to the bidder’s address correctly, I set the bid in pendingReturns back to the amount stored in the variable “amount” and return false.

```
1    function auctionEnd() public {
2        require(msg.sender == beneficiary && !end);
3        end = true;
4        msg.sender.transfer(highestBid);
5    }
```

In the line 2, I use a “require” to check whether the user who trigger the auctionEnd() function is the beneficiary. Besides, I also check whether the auction has already ended, which can avoid multiple calls. In the line 3 to 4, I set the Boolean variable “end” to true, which means the auction is end, and then transfer the highest bid to the beneficiary.

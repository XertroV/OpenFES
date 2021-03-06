# OpenFES Test Implementation Overview

## Intro

This is a really rough documentation of `MARKETCOIN` - the contract that will form
the first distributed [factum exchange](https://xk.io/wp/FactumExchange.md)
(though we shouldn't count our chickens). Don't 
worry about what I mean by factum at this stage.

`MARKETCOIN` enables users to trade on an automatic, fee free (virtually), peer to peer
exchange which deals in real money. Furthermore, the system is capable of expanding to 
more sophisticated and efficient (technically) markets when other currencies support 
cross-chain awareness (read: your block headers are showing). This enables SPV verification
internally and - at the cost of a slight dependence on the foreign chain - inter-network
conditionals.

While this dependence will be judged to unacceptably weaken the security of some networks 
(such as Bitcoin), some other currencies (especially smaller ones where the security risk 
is negligible) might find it worth the effort. The beauty is `MARKETCOIN` doesn't need
a network's consent to trade with it, and is completely transparent.

## OpenFES

OpenFES stands for the Open Factum Exchange Standard, which aims to become a standard by 
which blockchains interact to enable trustless exchange without deferring value to a token
as is done with many contemporary exchanges.

## Marketcoin History

Marketcoin was originally going to be a full blown cryptocurrency that was probably a little
ambitious. For posterity the original 'whitepaper' is located [here](https://github.com/XertroV/MarketcoinWhitepaper).

## Contracts

The following contracts are required:

* [`CHAINHEADERS`](chainheaders.pseudo)
* [`MERKLETRACKER`](merkletracker.pseudo)
* [`SPV`](spv.pseudo)
* [`MARKETCOIN`](marketcoin.pseudo)

## SPV

Three elements comprise the SPV check:

* chain headers 
* merkle tracker 
* SPV verification 

A user must provide:

* The relevant block header
* The relevant transaction hash
* The relevant missing parts of the merkle branch

`USER` sends missing parts of merkle branch to `MERKLETRACKER`
`SPV` will pull the block from `CHAINHEADERS` and extract the merkle root.
`SPV` will take the transaction hash and look through `MERKLETRACKER`'s database: 
(c.m = contract.memory)

	MERKLETRACKER:
	1. if not c.m[hash]: stop
	2. if c.m[hash] == 1: dosuccess() # indicates verified merkle root
	3. hash = sha256(c.m[hash] + hash)
	4. go to 1.

dosuccess(): (c.m is SPV's db in the following)

	add c.m[txhash] = blockhash
	
At this stage any other contract can see that the SPV contract has validated txhash exists 
in the block referenced by blockhash.

## Steps to verify transaction

If block hasn't been added to chainheaders
	
	(CHAINHEADERS, somefee, [blockheader])

If merkleroot hasn't been provided:
	
	(MERKLETRACKER, somefee, [0, merkleroot, blockhash])

Construct merkle branch proof
	
	(MERKLETRACKER, somefee, [1, hash1, hash2, lr ++ hash3, lr ++ hash4, ...])

	Where:
	lr is the byte indicating left/right of the hash
	And the merkle tree looks like:

	xxxxx xxxxx xxxxx xxxxx hash1 hash2 xxxxx xxxxx
	   xxxxx       xxxxx       _____       hash3
			 hash4                   _____
						 mklrt
						 
	Note: only hashes that are not already known about need to be added. In the example above, the 
	contract knows about mklrt already (added previously). If hash4 were also known  then only hashes
	1, 2 and 3 are needed.

Prove Alt Payment:

	(MARKETCOIN, somefee, [2, orderhash, alt_transaction, blockhash])
	
## Market

The market is structured slightly differently to most. This is because it is designed to be efficient
for the average person, not traders; it is because there are no spreads and almost no fees; it is
because it becomes equitable and helps remove the notion that price is without error.

Currently `MARKETCOIN` supports ignorant asymmetric exchange only. At a later time more complex
and elegant markets will be built.
	
	Sell ALT:
	(MARKETCOIN, somefee, [0, min_return, required_output, chain_entry, max_curr])
		value = pledge
		
	Sell ETR:
	(MARKETCOIN, somefee, [1, min_return, required_output, chain_entry])
		value = total order
		
	Prove Alt Payment:
	(MARKETCOIN, somefee, [2, ordermatch_index, alt_transaction, blockhash])
		
	Cancel Order:
	(MARKETCOIN, somefee, [3, orderhash, prev_orderhash])
	
	Push Pledge:
	(MARKETCOIN, somefee, [4, ordermatch_index])
		
The market is described in some detail in the old [Marketcoin Whitepaper](https://github.com/XertroV/MarketcoinWhitepaper).


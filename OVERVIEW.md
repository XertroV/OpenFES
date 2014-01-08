# OpenFES Test Implementation Overview

## SPV

Three elements comprise the SPV check:

* chain headers (CHS)
* merkle tracker (MTR)
* spv verification (SPV)

A user must provide:

* The relevant block header
* The relevant transaction hash
* The relevant missing parts of the merkle branch

USR sends missing parts of merkle branch to MTR
SPV will pull the block from CHS and extract the merkle root.
SPV will take the transaction hash and look through MTR's database: (c.m refers to MTR's db in the following)

	1. if not c.m[hash]: stop
	2. if c.m[hash] == 1: dosuccess()
	3. hash = sha256(c.m[hash] + hash)
	4. go to 1.

dosuccess(): (c.m is SPV's db in the following)

	add c.m[txhash] = blockhash
	
At this stage any other contract can see that the SPV contract has validated txhash exists in the block referenced by blockhash.

## Market

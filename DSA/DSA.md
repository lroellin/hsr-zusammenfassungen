---
typora-copy-images-to: ./Bilder
---

# DSA

# VSS Repetition

## Replication

Direct Replication: Originator Peer is the Responsible.

Indirect Replication: closest peer is Responsible

## Consistency

Consistency in a DHT is difficult

vDHT: for consistency (among others)

* Copy on write
* 2PC
* replication
* software transactional memory (STM)
* works for light churn
  * No locking, no timestamps
  * every update = new version
  * get latest version, check if all replica peers have latest version, if not wait and try again
  * put prepared with data and short TTL
    * if status is OK on all replica peers, go ahead,
    * otherwise, remove the data and go to step 1
  * put confirmed (don't send data, just remove the prepared flag)
* for heavy churn: API user needs to resolve

## Kademlia

Kademlia: if distance can be represented in m bits, bucket m will be used

Symmetrical routing paths

## Coins

Account-based (e.g. ETH):

* Global state stores a list of accounts with balances and code
* Transaction is valid if the sending account has enough balance
  * Balance on sender is deducted, new balance
* If the receiving account has code, the code runs and state may be changed
  * Signature must match sending account

![image-20190127094823962](Bilder/image-20190127094823962.png)

UTXO-based

* Every referenced input must be valid and not yet spent
* Total value of the inputs must equal (or exeed) the total value of the outputs
  * You always spend all outputs
* Transaction must have a signature matching the owner of the input for every input
  * Script determines if input is valid

![image-20190127095012370](Bilder/image-20190127095012370.png)


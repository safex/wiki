

# Safexcore release procedure


This document describes release procedure of [safexcore](https://github.com/safex/safexcore) project.

Development and bugfixes for current production release are performed on [develop](/github/safexcore/tree/develop) branch. Latest stable source code is on [master](/github/safexcore/tree/master) branch. Latest proof of concept development source code resides on [develop-poc](/github/safexcore/tree/develop-poc) branch.

Release procedure starts with freezing master branch and creating release branch, that is named *release-v.x.y.z* branch, where x, y and z are major, mid and minor version number, e.g. *release-v.0.1.2*.

This branch source code is further used for all tests. If there are citical bug and hotfixes are applied to this branch and in parralel to master branch.


## Build test

Static and dist *safexcore* source code build must be tested on the following platforms:

* Ubuntu 16.04
* Ubuntu 18.04
* MacOS
* Windows 10

Test must confirm build without errors for *safexd* node, C++ safex cli wallet *safex-wallet-cli* and C++ rpc wallet *safex-wallet-rpc*.

Build for other projects depending on safexcore must also be tested:
* [Safex Blockchain Explorer](https://github.com/safex/safex-blockchain-explorer) master branch build
* [Safex 1 Click Mining App](https://github.com/safex/safex_miner) master branch build
* [Safex Cash Orbiter Wallet](https://github.com/safex/wallet) master branch build

Build procedures for above mentioned projects are documented on their github pages.


## Core and unit tests

Perform core and unit tests on all platforms by building safexcore with `make static-all` and running tests in folder *build/release/tests* with `ctest -V` command.


## Stage network tests

Stage network is used for realistic production prerelease testing. 
There are two stage network nodes *68.183.73.230* and *138.68.79.110* and stagenet explorer is available on this [link](http://68.183.73.230). Stage network should be used for all integration tests. 


### Safexd node test

- Confirm version number with *version* node command
- Sync safexd node locally from scratch from existing network
- Sync locally second safexd node (on different port) from newly synced safexd node in the previous step
- Check blockchain height with `print_height`
- Check hard fork info with `hard_fork_info`
- Print transactions (//todo insert tx hashes here) with `print_tx <transaction_hash> +json` command
- Print block (//todo insert block hashes here) with `print_block <blockh_height>` command
- Print series of blockcs (//todo insert block numbers here) with `print_bc <height1> <height2>` commands
- Check safexd node mining by mining couple of blocks (`start_mining`, `stop_mining` commands)
- Check if key image (//todo insert here) is spent with (`is_key_image_spent <key_image>`)
- Print coinbase tx sum for number of blocks `print_coinbase_tx_sum <start_height> [<block_count>]`
- Print print_migrated_token_sum tx sum for number of blocks `print_migrated_token_sum <start_height> [<block_count>]`



### Safex cli wallet test

Prepare 3 stage net test wallets, *wallet01*, *wallet02* and *wallet03*. They should already be filled with some cash and token amount. Use them and wallet command line to perform following operations:

* Use previous existing wallet files and sync up one or more wallets
* Recreate one or more wallets from scratch using both seed and keys and sync them up
* Check viewkey and spendkey with `viewkey` and `spendkey` commands
* Check cash transfer
  * Send safex cash small amount from one wallet to another, check for dust handling
  * Send safex cash larger amount from one wallet to another
  * Send safex cash from one wallet to another and test long payment id
  * Send safex cash from one wallet to another and test short payment id
  * Test sending cash with reduced ring size
  * Check if receiving wallet has received payments with `payments <PID>` command
  * Check sending cash to interated address
  * Check sending cash to subaddress
* Check token transfer
  * Send safex token small amount from one wallet to another, check for dust handling
  * Send safex token larger amount from one wallet to another
  * Send safex token from one wallet to another and test long payment id
  * Send safex token from one wallet to another and test short payment id
  * Test sending token with reduced ring size
  * Check if receiving wallet has received token payments with `payments <PID>` command
  * Check sending token to interated address
  * Check sending token to subaddress
* Check if wallet is regularly updated with new blocks information
  * is correctly displaying balance updates, tx history, output updates?
* Test token migration (only authorized person with advanced wallet)
* Test sweep_unmixable operation (if needed, create some uncommon large output and try to spend it) 
* Check current available outputs with `unspent_cash_outputs`
* Check current token outputs with `unspent_token_outputs`
* Check local cli wallet transaction history
  * Are all transactions correctly displayed?
* Check particular transfer with `show_transfer <txid>` command
  * Check for inconsistencies
* Check blockchain rescan process with `rescan_bc` and `rescan_spent` commands
* Check balance with `balance`, `balance_cash`, `balance_token` commands
* Check proof if somebody has send transaction (`get_spend_proof`, `check_tx_proof`)
* Check transaction key for particular previous transaction with `get_tx_key`
 

### Safex RPC wallet test

Use (//todo insert JS script link) for RPC wallet testing

Repeat all previous cli wallet tests with RPC wallet, using RPC scripts. 


### Blockchain explorer test

Check stagenet blockchain explorer.

* Blocks info getting updated regularly
* Check information for one block
* Check transaction info display
  * Check if it is possible to decode outputs of the transaction
  * Check if it is possible to prove sending of the transaction


### Mining tests

#### Cpu minter test

Basic test of Safex 1 Click Mining App.
    
* Run and sync one click CPU miner
* Check token and cash initial balance
* Test mining with one click CPU miner
* Observe cash balance updates while mining is in probress
* Try to send cash with one click CPU miner and check for balance updates
* Try to send tokens with one click CPU miner and check for balance updates
    

#### GPU Mining test

If there were changes related to hash mining algorithm, GPU mining test is a must.


Perform GPU solo mining test, information about build procedures are available [here](https://github.com/safex/safexcore/blob/release-v0.1.2/Mining.md).

* Build monero-stratum with latest safexcore build
* Build latest xmrig-nvidia and test GPU mining with Nvidia GPU
* Build latest xmr-stak and test GPU mining with AMD GPU


### Safex Cash Orbiter Wallet





## Release

After all tests pass successfully, new github release will be created. It should contain Windows, Ubuntu 18.04, Ubuntu 16.04 and OSX binaries for node, command line wallet and rpc wallet.








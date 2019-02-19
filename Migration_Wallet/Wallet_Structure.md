> The data structure used in the github.com/safex/safex_wallet repo
> This is the wallet from August 2017-Present February 2019 
> That includes the migration system

The safex wallet uses a json structure to store information about a users wallet

This includes all keys and other meta information needed for the migration process

Let's dive in.

### Example Safex Wallet File

```
    {
      "version": "1",
      "keys": [
          {
                "public_key": "1A22fwGJToeW342Qd9awfwaf58CS1STZX8Ag",
                "private_key": "5ThWfzAySUzue1Ww64QYXvjafwafawfawMLxJHEyLUgvPuU1",
                "safex_bal": 0,
                "btc_bal": 0,
                "pending_safex_bal": 0,
                "pending_btc_bal": 0,
                "migration_data": {
                  "safex_keys": {
                    "spend": {
                      "sec": "eafc46457009932cecdeadbeef65a3a3ab62deadbeef629688795b0c",
                      "pub": "1b440abc110acee1deadbeef99cd20c53ccee0deadbeef25be2c967eb41f84939"
                    },
                    "view": {
                      "sec": "fec1f271deadbeefb4b7b77c6deadbeef73e563717c5772eadbeefa540e07",
                      "pub": "403a6453c8f498f6deadbeef31eb8a88d8dbfeadbeef1eadbeef8b2c377"
                    },
                    "public_addr": "Safex5z23o444TUvAibUzaiKAZqf3AWFAawfawfWFAcFNRzuQZ6yEiip5oQj6Wsd2mctUkQ7vSrErgn6f1b",
                    "checksum": "552ba422"
                  }
                },
                "migration_progress": 3,
                "archived": false
          },
          {
                "public_key": "1124gasfQgXQSwafwafwFFdpbyqSgh98M",
                "private_key": "522dcHjBxPbAawfawfhEEJERJRSJRTJEUwagwGGGzv7vG5qEKu",
                "safex_bal": 0,
                "btc_bal": 0,
                "pending_safex_bal": 0,
                "pending_btc_bal": 0,
                "archived": false,
                "label": "Enter your label here"
           },
          ...
       ],
       "safex_keys": [
           {
             "spend": {
               "sec": "98a6deadbeeff020deadbeef03613deadbeefff47adedeadbeef0b38ff0305",
               "pub": "c357deadbeeff7a7ba08deadbeef75865e2670aaf0bbd9deadbeef05ea0714e13d0ef"
             },
             "view": {
               "sec": "5e4f2deadbeef9cdc2c98c477bdeadbeef8b9ae03c1deadbeef362c0002",
               "pub": "4603127deadbeefd0258854deadbeeff12852943f1deadbeefdc764c6c206dd"
             },
             "public_addr": "Safex5555SHSERheejZknnNFAWHAHEmVA3BCtwKxCcvSSwkChMmGasGEaGEGD4TxWt3nLHEHSEHSEHzgRmew2nHrf89zW5Lgm3f",
             "checksum": "36d3ae9a"
           },
           ...
        ]
    }
```


### Migration Progress

     "migration_progress": 3, //can be a 1, 2, 3, or 4
                         ^^^^^

1. Acknowledge the terms of the process
2. Setting the first half of the Safex Address (safex blockchain)
3. Setting the second half of the Safex Address (safex blockchain)
4. Final stage where the Safex Address is set, and user can send their Safe Exchange Coins to be credited to the set 
Safex Address (from bitcoin to safex blockchain migration process)

Hitting `reset` on the migration interface will set the `migration_progress` to 0 and cause the UI to load
the process from the beginning.


* **Version** - the wallet version number, so that in future iterations can be coded to identify which 
version of the wallet file isbeing interacted with.
* **Keys** - An array of `Secp256k1` keys used for storing `Bitcoins` and `Safe Exchange Coins` which 
adhere to the `Omni Protocol`
    * **Keys Object** - This stores the `public` and `private` keys as well as `meta information` for the wallet
    functionality, in addition to the `migration data`.
        * **public_key** - this is a `bitcoin` public key in the `address` format.
        * **private_key** - this is a `wif` format private key for `bitcoin` or `safe exchange coin`.
        * **safex_bal** - this field is used internally to store the balance that is displayed. 
        It is never written to the actual wallet file. This would represent the `Safe Exchange Coin` Balance.
        * **btc_bal** - this field is used internally to store the balance that is displayed. 
        It is never written to the actual wallet file. This would represent the `Bitcoin` Balance.
        * **pending_safex_bal** - this field is used to display the net pending balance for `Safe Exchange Coin`.
        It will take outgoing and incoming transactions and display the net amount.
        * **pending_btc_bal** - this field is used to display the net pending balance for `Bitcoin`.
        It will take outgoing and incoming transactions and display the net amount.
        * **migration_data** - the migration field stores an object for the migration process. This object
        is the active information that was last set. So if someone started the migration process this is
        where the keys are stored. 
            * **safex_keys** - this object is part of the `migration_data` object. This is the object that
            stores the active `safex blockchain` key used in the migration process. The keys are generated 
            using the ed25519 curve.
                * **spend** - this is the `spend key pair` as part of the `cryptonote` architecture, 
                the `safex blockchain` uses a two key system, the spend and view keys. The `spend key` 
                is used as part of spending inputs that were discovered by the view key.
                    * **sec** - the secret part of the key pair.
                    * **pub** - the public part of the key pair.
                * **view** - this is the `view key pair` as part of the `cryptonote` architecture, 
                the `safex blockchain` uses a two key system, the spend and view keys. The `view key`
                is used to scan the blockchain and retrieve outputs that belong to you as inputs for 
                future transactions.
                    * **sec** - the secret part of the key pair.
                    * **pub** - the public part of the key pair.
                * **public_addr** - the `public address` is a combination of the `public view key` and the 
                `public spend key` hashed with checksums and passed through `cnBase58` which is a variation 
                on the base58 function.
                * **checksum** - the keccak256 of the address prefix + spend pub key + view pub key.
            * **migration_progress** - this is a number that represents where the user is present in during
            the migration process. [Read the detailed breakdown of the phases.](#migration-progress)
            * **archived** - if this is true, then the address is hidden in the `archive` section and out of
            site from the main view of the list of addresses.
            * **label** - if a label had been set, then this is the field that will hold the text that
            a user can use to give a description to the address.
* **safex_keys** - this is an array that stores all `safex keys` generated throughout the life of the wallet.
These are safex keys that are collected even through multiple iterations of the migration process, including
across different addresses.
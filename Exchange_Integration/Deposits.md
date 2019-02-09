> Guidance for Exchange Integration Deposit Acceptance


**Start Executables**

1. `./safexd --restricted-rpc`

(the restricted rpc flag forces your node to only permit basic wallet functionality)

2. `./safex-wallet-rpc --rpc-bind-port 17405 --wallet-file <mywallet> --prompt-for-password`

At first you must start the daemons, one is the connection to the safex blockchain and the second allows you to programmatically manage wallet file actions such as checking balance, generating addresses for deposit, and spending funds.


You should replace `<mywallet>` with a wallet file generated with the `./safex-wallet-cli` and the password will be the one used when that wallet file was created.


Next Step is programming and setting up your system.

Since the safex blockchain uses the CryptoNote method for secure payments, therefore you need a way to correlate payments internally. We do so with a Payment ID.

A payment ID is a 16 character hex string. Once generated you must pad it with 48 0's until it is 64 characters long.

Then, in your database of Users you must store this unique hexadecimal string. Each User must have its own unique payment ID.


In order to generate a `payment ID` you have two options:

1. You can manually generate the `16 character hex strings`, remember that they must be unique.

2. The `safex-wallet-rpc` can generate them for you. 


First way (you generate your own payment ids):

1. Generate a `payment ID` for the user. 

	For example: this could be the id# of the user in the database converted to hex demonstrated in python snippet below:

		 > PID = binascii.hexlify((user_id.to_bytes(32, 'little'))).decode('utf-8')

2. Store this `PID` with the `User`

3. Now we will create the `Integrated Address` for the `User`.

4. `Trim` the last `48 characters` which are `0's` from the `PID`.

5. `Call` to the `safex-wallet-rpc` with the following command using the `trimmed PID` as the `payment_id` parameter:

	    > curl -X POST http://127.0.0.1:17405/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"make_integrated_address","params":{"standard_address":"<YOUR SAFEX 		ADDRESS", "payment_id":"<TRIMMED PID>"}}' -H 'Content-Type: application/json'

6. `Store` the responded `Integrated Address` to the `User`.

7. You can verify the stored payment id with the response received that the two are matching, what is stored in the database and the was provided from the safex-wallet-rpc

	
Now Verifying received deposits.
		Now in order to verify that a deposit was received we recommend using the `get_bulk_payments` method.
		
You will supply the starting `block height` from which to scan. After running the `get_bulk_payments` method you should record the last mined top block.

1. First thing that we will do is run the `get_bulk_payments` method. 
        
        > curl -X POST http://127.0.0.1:17405/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"get_bulk_payments","params":{"min_block_height":0}}' -H 			'Content-Type: application/json'
		
2. Next we run `get_height` to record the top block that we scanned:
		
		> curl -X POST http://127.0.0.1:17405/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"get_height","params":{}}' -H 			'Content-Type: application/json'

3. Then you should subtract 1 from the received block height since the top block could be a block in progress of mining. We don't want to miss any blocks, this 			method will ensure we are scanning each and every block for transactions. This order is essential because if you are scanning a large distance of the chain, 			then blocks will be constantly produced while you are still processing transactions.

	The response from `get_bulk_payments` will look like this:
```
{
  "id": "0",
  "jsonrpc": "2.0",
  "result": {
    "payments": [{
      "address": "Safex5yssKFcekLCkgoUGXW4uiyABSgMidrDyyJFBNFxMT8XXt949iqFX8mx1WFiAWBZoSf7ywx8hg7MpCmJTjss1oA4ubauQzp2E",
      "amount": 500000000000,
      "block_height": 45056,
      "payment_id": "a1b2c5d4e5e61236000000000000000000000000000000000000000000000000",
      "subaddr_index": {
        "major": 0,
        "minor": 0
      },
      "token_amount": 0,
      "token_transaction": false,
      "tx_hash": "fd8326394ed0945b46a41474ce0b1deba6fb6c44d09ab22222918abf85eaeb78",
      "unlock_time": 0
    },{
      "address": "Safex5yssKFcekLCkgoUGXW4uiyABSgMidrDyyJFBNFxMT8XXt949iqFX8mx1WFiAWBZoSf7ywx8hg7MpCmJTjss1oA4ubauQzp2E",
      "amount": 0,
      "block_height": 45056,
      "payment_id": "a1b2c5d4e5e61236000000000000000000000000000000000000000000000000",
      "subaddr_index": {
        "major": 0,
        "minor": 0
      },
      "token_amount": 100000000000,
      "token_transaction": true,
      "tx_hash": "702e59747fb0f24c78e169930542193e21a76136e0fc0be49036c4c77a51bc76",
      "unlock_time": 0
    }
      ....
    ]
  }
}
```

The above response has an object: results, and inside we have the payments object that is an array of transactions from all blocks scanned. 
	
5. Now we iterate over the `payments` and extract "payment_id" and "tx_hash" and check our payments table in the database that we have not already processed this "tx_hash". We should store payment_id, tx_hash, block_height, amount, and token or cash transaction.

		5. a. To check if the transaction is token or cash you check the "token_transaction" field. If it is `true` then this is a token transaction.
	
		5. b. If the token_transaction is `true` then the minimum amount is 10000000000 or 10 billion. 

		5. c. If the token_transaction is `false` then it is a safex cash transaction.

6. If we do not have a collision in the payments table, then we can update our User balance by matching the "payment_id" from the payment in question.

7. Repeat for all payments in the array.

8. Run this regularly at least each two minutes to be most up to date with your Users deposits.		


Now Confirming Deposits:
	It is recommended to allow several blocks to pass before considering a transaction fully confirmed and crediting a User on your exchange. We recommend 15 confirmations. This means you should have the difference of 15 blocks between the transactions block_height and the network's block_height.

	1. Call to the payments table, scan payments by block_height and do arithmetic between current block_height and the block_height of the transaction.

	2. If the difference is greater than 15 then you should be able to safely credit the User and consider the transaction to be final.



Using the safex-wallet-rpc to generate payment ids:
	1. Generate a payment ID and Integrated Address for the User. 
		Call to the safex-wallet-rpc with the following command using the trimmed PID as the "payment_id" parameter:
		curl -X POST http://127.0.0.1:17405/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"make_integrated_address","params":{"standard_address":"<YOUR SAFEX 		ADDRESS"}}' -H 'Content-Type: application/json'

	In response you will received:
		{
 "id": "0",
 "jsonrpc": "2.0",
 "result": {
   "integrated_address": "SFXti9o1apCRhgUEU4FVJjBkNS9sarNpbexP6YfZgDYv3bcSVwCZtm9PWnpkoRiifC3uMQJS9ihFmNTbUXr2eWgY7LUMiNBLJ8tYNMHdD3hE6d",
   "payment_id": "0f06ee25bb89a108"
 }
}

2. Pad the responded "payment_id" with 48 0's so that it is 64 characters long and store this for the User along with the Integrated Address.
	

	


## Table of contents


# What is Pump.Fun API?

**A 3rd-Party (unofficial) API for Pump.Fun**. We provide HTTPS endpoints that allow you to programmatically trade on, and gather data from, the Pump.fun bonding curve. 

Pump.Fun API is designed to be easy to use, so you can start building in minutes: there are no libraries to install and nothing to configure. 

We're excited to see your trading bots, websites, or any other applications you integrate with Pump.Fun! Connect with us in [Telegram](https://t.me/+60jIercvZa4zZmZl) to ask questions or show off what you're building. If you build something cool we might feature your project on this page!

# Local Trading API Docs

To get a transaction for signing locally and sending with a custom RPC, send a POST request to 

`https://astgo.vip/api/PumpTrade-Local`

Your request body must contain the following options:

- `publicKey`: Your wallet public key
- `action`: "BUY" or "SELL"
- `tokenAddress`: The contract address of the token you want to trade (this is the text after the '/' in the pump.fun url for the token.)
- `amount`: The amount of SOL or tokens to trade. If selling, amount can be a percentage of tokens in your wallet (ex. amount: "100%")
- `slippage`: The percent slippage allowed
- `priorityFee`: Amount to use as priority fee

If your parameters are valid, you will receive a serialized transaction in response. See the example below for how to send this transaction with Web3.js.

## Examples


```js
import { VersionedTransaction, Connection, Keypair } from '@solana/web3.js';
import bs58 from "bs58";

const RPC_ENDPOINT = "Your RPC Endpoint";
const web3Connection = new Connection(
    RPC_ENDPOINT,
    'confirmed',
  );

async function sendPortalTransaction(){
      const response = await fetch(`https://astgo.vip/api/trade-local`, {
          method: "POST",
          headers: {
              "Content-Type": "application/json"
          },
          body: JSON.stringify({
              "publicKey": "your-public-key",  // Your wallet public key
              "action": "BUY",                 // "BUY" or "SELL"
              "tokenAddress": "token-ca-here",         // contract address of the token you want to trade
              "amount": "10",                  // amount of SOL or tokens
              "slippage": "5",                   // percent slippage allowed
              "priorityFee": "0.0001",          // priority fee
          })
      });
      if(response.status === 200){ // successfully generated transaction
          const result = await response.json();
          const data = Buffer.from(result.serializedTransaction, 'base64');
          const tx = VersionedTransaction.deserialize(new Uint8Array(data));
          const signerKeyPair = Keypair.fromSecretKey(bs58.decode("your-wallet-private-key"));
          tx.sign([signerKeyPair]);
          const signature = await web3Connection.sendTransaction(tx)
          console.log("Transaction: https://solscan.io/tx/" + signature);
      } else {
          console.error(response.statusText); // log error
      }
}

sendPortalTransaction();
```

# Fees

## Trading API

We take a 0.9% fee on each trade. (Note: the fee is calculated before any slippage, so the actual fee amount may not be exactly 0.9% of the final trade value.)

This does not include Solana network fees, or any fees charged by the Pump.fun bonding curve.

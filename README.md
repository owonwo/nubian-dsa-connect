# NUBIAN DSA Connect ![Build Status](https://nubian-frontend.netlify.app/static/media/logo.cf0483d3.svg)

The official DSA SDK for JavaScript, available for browsers and Node.js backends.

## Installation

To get started, install the DSA Connect package from npm:

```bash
npm install nubian-dsa-connect
```

For browsers, via jsDelivr CDN:

```html
<script src="https://cdn.jsdelivr.net/npm/nubain-dsa-connect@latest/dist/index.bundle.min.js"></script>
```

### Usage

To enable web3 calls via SDK, instantiate [web3 library](https://github.com/ChainSafe/web3.js#installation)

```js
// in browser
if (window.ethereum) {
  window.web3 = new Web3(window.ethereum)
} else if (window.web3) {
  window.web3 = new Web3(window.web3.currentProvider)
} else {
  window.web3 = new Web3(customProvider)
}
```

```js
// in nodejs
const Web3 = require('web3')
const DSA = require('nubian-dsa-connect')
const web3 = new Web3(new Web3.providers.HttpProvider(ETH_NODE_URL))
```

Now instantiate DSA with web3 instance.

```js
// in browser
const dsa = new DSA(web3)

// in nodejs
const dsa = new DSA({
  web3: web3,
  mode: 'node',
  privateKey: PRIVATE_KEY,
})
```

## Setting up DSA Accounts

Every user needs to create Smart Account to interact with DeFi Protocols seamlessly; this allows developers to build extensible use-cases with maximum security and composability. You can also create multiple account for a single address.

- Create Smart Account - `build()`
- Fetch Smart Accounts - `getAccounts()`
- Set Smart Account - `setInstance()`

### build()

Create a DSA Account. If the account is already created, you can use the `setInstance` method to activate a paricular DSA account and start casting spells.

```js
// in async functions
await dsa.build()

// or
dsa.build().then(console.log)
```

The build method also accepts an optional parameters as shown below:

```js
dsa.build({
  gasPrice: gasPrice // estimated gas price
  origin: origin,
  authority: authority,
})
```

> View this [Gist](https://gist.github.com/thrilok209/8b19dbd8d46b2805ab8bb8973611aea2) for estimation of gas price

| **Parameter** | **Type**        | **Description**                                                                                                                                                                |
| ------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `gasPrice`    | _string/number_ | The gas price in gwei. Mostly used in Node implementation to configure the transaction confirmation speed.                                                                     |
| `origin`      | _address_       | The address to track the origin of transaction. Used for analytics and affiliates.                                                                                             |
| `authority`   | _address_       | The DSA authority. The address to be added as authority.                                                                                                                       |
| `from`        | _address_       | The account with which you want to create your DSA. This is helpful to create DSA for other addresses.                                                                         |
| `nonce`       | _string/number_ | Nonce of your sender account. Mostly used in Node implementation to send transaction with a particular nonce either to override unconfirmed transaction or some other purpose. |

The method returns the transaction hash.

This creates a uniquely numbered Smart Account which acts as a proxy to interact with verified DeFi protocols and each DSA has a unique ethereum address.

### getAccounts()

Fetch all the accounts owned by an ethereum address by calling `getAccounts()`.

```js
// in async functions
await dsa.getAccounts(address)

// or
dsa.getAccounts(address).then(console.log)
```

| **Parameter** | **Type**  | **Description**      |
| ------------- | --------- | -------------------- |
| `address`     | _address_ | An ethereum address. |

The method returns an array of objects with all the DSA accounts where `address` is authorised:

```js
[
  {
      id: 52, // DSA ID
      address: "0x...", // DSA Address
      version: 1 // DSA version
  },
  ...
]
```

### setInstance()

Be sure to configure global values by calling `setInstance()`. You can get the id by calling `getAccounts()`. The configured account will be used for all subsequent calls.

```js
dsa.setInstance(dsaId) // DSA ID
```

| **Parameter** | **Type** | **Description**                |
| ------------- | -------- | ------------------------------ |
| `dsaId`       | _Number_ | DSA ID to be used for casting. |

The method returns an array of objects with all the DSA accounts where `address` is authorised:

## Casting Spells

**Spells** denotes a sequence of connector functions that will achieve a given use case. Spells can comprise of any number of functions across any number of connectors.

With this SDK, performing DeFi operations on your dapp consists of creating a `spells` instance to add transactions. Here is where you can initiate complex transactions amongst different protocols.

Create an instance:

```js
let spells = dsa.Spell()
```

Add **spells** that you want to execute. Think of any actions, and by just adding new SPELLS, you can wonderfully CAST transactions across protocols. Let's try to execute the following actions:

- Deposit 1 ETH to Venus protocol
- Borrow 100 DAI
- Deposit borrowed DAI on Venus

```js
- Deposit

spells.add({
  connector: "VenusV2",
  method: "deposit",
  args: [
    "0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee",
    "1000000000000000000", // 1 ETH (10^18 wei)
    0,
    0
  ]
})

- Borrow

spells.add({
  connector: "VenusV2",
  method: "borrow",
  args: [
    "DaiAddress",
    "100000000000000000000" // 100 Dai (10^18 wei),
    0,
    0
  ]
})

- Deposit Dai

spells.add({
  connector: "VenusV2",
  method: "deposit",
  args: [
    "DaiAddress",
    "100000000000000000000" // 100 Dai (10^18 wei),
    0,
    0
  ]
})
```

**Note** - Make sure, your smart account have the equivalent BNB balance before executing the above actions.

At last, cast your spell using `cast()` method.

```js
// in async functions
let transactionHash = await spells.cast()

// or
spells.cast().then(console.log) // returns transaction hash
```

You can also pass an object to send **optional** parameters like sending ETH along with the transaction.

```js
spells.cast({
  gasPrice: web3.utils.toWei(gasPrice, 'gwei'), // in gwei, used in node implementation.
  value: '1000000000000000000', // sending 1 BNB along the transaction.
  nonce: nonce,
})
```

| **Parameter (optional)** | **Type**        | **Description**                                                                                                                                                                |
| ------------------------ | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `gasPrice`               | _string/number_ | The gas price in gwei. Mostly used in Node implementation to configure the transaction confirmation speed.                                                                     |
| `value`                  | _string/number_ | Amount of BNB which you want to send along with the transaction (in wei).                                                                                                      |
| `nonce`                  | _string/number_ | Nonce of your sender account. Mostly used in Node implementation to send transaction with a particular nonce either to override unconfirmed transaction or some other purpose. |

This will send the transaction to blockchain in node implementation (or ask users to confirm the transaction on web3 wallet like Metamask).

## Connectors

You can see the list of connectors [here](src/addresses) & [here](https://github.com/Open-Currency-Collective/nubian-dsa-connect/blob/main/README.md)

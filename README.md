# Dai.js Wiki

# Dai.js Documentation

**Navbar:** 

- Getting-started
- Maker
- CDP
- Price
- cdp-type-parameters
- Exchange
- System-status
- Events
- Transaction
- Accounts
- Proxy-service
- Source-code-setup
- New-service

# Dai.js

**Dai.js** is a JavaScript library that makes it easy to build applications on top of MakerDAO's platform of smart contracts. You can use Maker's contracts to open Collateralized Debt Positions (CDPs), withdraw loans in Dai, trade tokens on OasisDEX, and more.
The library features a pluggable, service-based architecture, which allows users maximal control when integrating the Maker functionality into existing infrastructures. It also includes convenient configuration presets for out-of-the-box usability and support for both front-end and back-end applications.
Maker's entire suite of contracts will eventually be accessible through this library—including the DAO governance and the upcoming multi-collateral release—but functionality is limited in the current alpha version to the following areas:

- Opening and closing CDPs
- Locking and unlocking collateral
- Withdrawing and repaying Dai
- Automated token conversions
- Token contract functionality for WETH, PETH, MKR, Dai, and ETH
- Buying and selling MKR and Dai with built-in DEX integration

For example code that consumes the library, check out [this repository](https://github.com/makerdao/integration-examples).


# **Getting Started**
## **Installation**

Install the package with npm:
`npm install @makerdao/dai`
Once it's installed, import the module into your project as shown on the right.


    import Maker from '@makerdao/dai';
    // or:
    const Maker = require('@makerdao/dai');

**UMD**
This library is also accessible as a [UMD module](https://github.com/umdjs/umd).

    <script src="./dai.js" />
    
    <script>
    Maker.create('http',{
        privateKey: YOUR_PRIVATE_KEY,
        url: 'https://kovan.infura.io/v3/YOUR_INFURA_PROJECT_ID'
    })
        .then(maker => { window.maker = maker })
        .then(() => maker.authenticate())
      .then(() => maker.openCdp())
      .then(cdp => console.log(cdp.id));
    </script>

This library is also accessible as a [UMD module](https://github.com/umdjs/umd).


## **Quick Example**

Using the [Maker](https://makerdao.com/documentation/#maker) and [Cdp](https://makerdao.com/documentation/#cdp) APIs, the code shown on the right opens a cdp, locks 0.25 ETH into it, draws out 50 Dai, and then logs information about the newly opened position to the console.


    import Maker from '@makerdao/dai';
    
    async function openLockDraw() {
        const maker = await Maker.create("http", {
            privateKey: YOUR_PRIVATE_KEY,
            url: 'https://kovan.infura.io/v3/YOUR_INFURA_PROJECT_ID'
        });
    
      await maker.authenticate();
      const cdp = await maker.openCdp();
    
      await cdp.lockEth(0.25);
      await cdp.drawDai(50);
    
      const debt = await cdp.getDebtValue();
      console.log(debt.toString); // '50.00 DAI'
    }
    
    openLockDraw();


# **Maker**
## **Presets**

When instantiating a `Maker` object, you pass in the name of a configuration preset and an options hash.
Available presets are listed below, with the required options for each shown on the right.

- `'browser'`
  - Use the provider from the browser (e.g. MetaMask).
- `'http'`
  - Use Web3.providers.HttpProvider.
- `'test'`
  - Use a local testnet (e.g. Ganache) running at `http://127.0.0.1:2000`, and sign transactions using testnet-managed keys.
  - 
    const makerBrowser = await Maker.create('browser');
    
    const makerHttp = await Maker.create('http', {
      privateKey: YOUR_PRIVATE_KEY,
      url: 'https://kovan.infura.io/v3/YOUR_INFURA_PROJECT_ID'
    });
    
    const makerTest = await Maker.create('test');


## **Options**


- `privateKey`
  - The private key used to sign transactions. If not provided, the first account available from the Ethereum provider will be used.
- `url`
  - The URL of the node to connect to. Only used when `provider.type` is `"HTTP"`.
- `provider.type`
  - `"TEST"`: connect to a local test blockchain at 'http://127.1:2000'
  - `"HTTP"`: connect to an Ethereum node via any arbitrary url
- `web3.statusTimerDelay`
  - Number in milliseconds that represents how often the blockchain connection and account authentication is checked. This allows the library to move out of an authenticated or connected state when it discovers it no longer has access to unlocked accounts, or can no longer connect to a node.
  - Default value: 5000 (5 seconds)
- `web3.transactionSettings`
  - Object containing transaction options to be applied to all transactions sent through the library.
  - Default value: `{ gasLimit: 4000000 }`
- `web3.confirmedBlockCount`
  - Number of blocks to wait after a transaction has been mined when calling `confirm`. See [transactions](https://makerdao.com/documentation/#transactions) for further explanation. Defaults to 5.
- `log`
  - Set this to `false` to reduce the verbosity of logging.
  
    const maker = await Maker.create('http', {
      privateKey: YOUR_PRIVATE_KEY, // '0xabc...'
      url: 'http://some-ethereum-rpc-node.net',
      provider: {
        type: 'HTTP', // or 'TEST'
        network: 'kovan'
      },
      web3: {
        statusTimerDelay: 2000,
        confirmedBlockCount: 8
        transactionSettings: {
          gasPrice: 12000000000
        }
      },
      log: false
    });


## **authenticate**

After creating your Maker instance, and before using any other methods, run this. It makes sure all services are initialized, connected to any remote API's, and properly authenticated.


    await maker.authenticate();


## **service**

**Params:** service (string)

- **Returns:** service object

The service function can be used to access services that were injected into your instance of `maker`. See the service documentation sections below.

    const priceService = maker.service('price');


## **openCdp**

**Params:** none

- **Returns:** promise (resolves to new CDP object once mined)

`maker.openCdp()` will create a new CDP, and then return the CDP object, which can be used to access other CDP functionality.
The promise will resolve when the transaction is mined.


    const newCdp = await maker.openCdp();


## **getCdp**

**Params:** CDP ID (integer)

- **Returns:** promise (resolves to CDP object)

`maker.getCdp(id)` creates a CDP object for an existing CDP. The CDP object can then be used to interact with your CDP.


    const cdp = await maker.getCdp(614);


## **Units**

Methods that take numerical values as input can also take instances of token classes that the library provides. These are useful for managing precision, keeping track of units, and passing in wei values.
Most methods that return numerical values return them wrapped in one of these classes.
There are two types:

- **currency units**, which represent an amount of a type of currency
- **price units**, which represent an exchange rate between two currencies.

The classes that begin with `USD` are price units; e.g. `USD_ETH` represents the price of ETH in USD.
Useful instance methods:

- **toNumber**: return the raw JavaScript value. This may fail for very large numbers.
- **toBigNumber**: return the raw value as a [BigNumber](https://github.com/MikeMcl/bignumber.js).
- **isEqual**: compare the values and symbols of two different instances.
- **toString**: show the value in human-readable form, e.g. "500 USD/ETH".


    import Maker from '@makerdao/dai';
    const {
      MKR,
      DAI,
      ETH,
      WETH,
      PETH,
      USD_ETH,
      USD_MKR,
      USD_DAI
    } = Maker;
    
    // These are all identical:
    
    // each method has a default type
    cdp.lockEth(0.25);
    cdp.lockEth('0.25');
    
    // you can pass in a currency unit instance
    cdp.lockEth(ETH(0.25));
    cdp.lockEth(ETH.wei(250000000000000000));
    
    // you can pass the unit as an options argument
    cdp.lockEth(0.25, { unit: ETH });
    cdp.lockEth(250000000000000000, { unit: ETH.wei });
    
    const eth = ETH(5);
    eth.toString() == '5.00 ETH';
    
    const price = USD_ETH(500);
    price.toString() == '500.00 USD/ETH';
    
    // multiplication handles units
    const usd = eth.times(price);
    usd.toString() == '2500.00 USD';
    
    // division does too
    const eth2 = usd.div(eth);
    eth2.isEqual(eth);
    


# **CDP**

Now that you've used `maker` to either open a new CDP or retrieve an existing one, you can use the returned `cdp` object to call functions on it.


## **Properties**

`**.id**`
This is the ID of the CDP object. You can pass this ID to `[Maker.getCdp](https://makerdao.com/documentation/#getcdp)`.


## **getDebtValue**


- **Params:** [currency unit](https://makerdao.com/documentation/#units) (optional)
- **Returns:** promise (resolves to the amount of outstanding debt)

`cdp.getDebtValue()` returns the amount of debt that has been borrowed against the collateral in the CDP. By default it returns the amount of Dai as a [currency unit](https://makerdao.com/documentation/#units), but can return the equivalent in USD if the first argument is `Maker.USD`.


    const daiDebt = await cdp.getDebtValue();
    const usdDebt = await cdp.getDebtValue(Maker.USD);



## **getGovernanceFee**


- **Params:** [currency unit](https://makerdao.com/documentation/#units) (optional)
- **Returns:** promise (resolves to the value of the accrued governance fee in USD)

`cdp.getGovernanceFee()` returns the value of the accrued governance fee. By default it returns the amount of MKR as a [currency unit](https://makerdao.com/documentation/#units), but can return the equivalent in USD if the first argument is `Maker.USD`.
*Note: this is often referred to as the* `*Stability Fee*`*, even though technically the* `*Stability Fee*` *is the fee that is paid in Dai, and the* `*Governance Fee*` *is the fee that is paid in MKR. But since fees are only paid in MKR in Single-Collateral Dai, and only paid in Dai in Multi-Collateral Dai, the fee in Single-Collateral Dai is often referred to as the* `*Stability Fee*` *to be consistent with the term that will be used in Multi-Collateral Dai and to avoid unduly confusing regular users.*


    const mkrFee = await cdp.getGovernanceFee();
    const usdFee = await cdp.getGovernanceFee(Maker.USD);



## **getCollateralizationRatio**

**Params:** none

- **Returns:** promise (resolves to the collateralization ratio)

`cdp.getCollateralizationRatio()` returns the USD value of the collateral in the CDP divided by the USD value of the Dai debt for the CDP, e.g. `2.5`.


    const ratio = await cdp.getCollateralizationRatio();



## **getLiquidationPrice**

**Params:** none

- **Returns:** promise (resolves to the liquidation price)

`cdp.getLiquidationPrice()` returns the price of Ether in USD that causes the CDP to become unsafe (able to be liquidated), all other factors constant. It returns a `USD_ETH` [price unit](https://makerdao.com/documentation/#units).

    const ratio = await cdp.getLiquidationPrice();



## **getCollateralValue**


- **Params:** [currency unit](https://makerdao.com/documentation/#units) (optional)
- **Returns:** promise (resolves to collateral amount)

`cdp.getCollateralValue()` returns the value of the collateral in the CDP. By default it returns the amount of ETH as a [currency unit](https://makerdao.com/documentation/#units), but can return the equivalent in PETH or USD depending on the first argument.


    const ethCollateral = await cdp.getCollateralValue();
    const pethCollateral = await cdp.getCollateralValue(Maker.PETH);
    const usdCollateral = await cdp.getCollateralValue(Maker.USD);


## **isSafe**

**Params:** none

- **Returns:** promise (resolves to boolean)

`cdp.isSafe()` returns true if the cdp is safe, that is, if the USD value of its collateral is greater than or equal to USD value of the its debt multiplied by the liquidation ratio.


    const ratio = await cdp.isSafe();



## **enoughMkrToWipe**

**Params:** amount of Dai to wipe

- **Returns:** promise (resolves to boolean)

`cdp.enoughMkrToWipe(dai)` returns true if the current account owns enough MKR to wipe the specified amount of Dai from the CDP.


    const enoughMkrToWipe = await cdp.enoughMkrToWipe(10000000000000000000, DAI.wei);



## **lockEth**


- **Params:** amount to lock in the CDP, in units defined by the price service.
- **Returns:** promise (resolves to `[transactionObject](https://makerdao.com/documentation/#transactions)` once mined)

`cdp.lockEth(eth)` abstracts the token conversions needed to lock collateral in a CDP. It first converts the ETH to WETH, then converts the WETH to PETH, then locks the PETH in the CDP.
*Note: this process is not atomic, so it's possible for some of the transactions to succeed but not all three. See* [*Using DsProxy*](https://makerdao.com/documentation/#proxy-service) *for executing multiple transactions atomically.*


    return await cdp.lockEth(10000000000000000000, ETH.wei);
    // or equivalently
    return await cdp.lockEth(100, ETH);



## **drawDai**


- **Params:** amount to draw (in Dai, as string)
- **Returns:** promise (resolves to `[transactionObject](https://makerdao.com/documentation/#transactions)` once mined)

`cdp.drawDai(dai)` withdraws the specified amount of Dai as a loan against the collateral in the CDP. As such, it will fail if the CDP doesn't have enough PETH locked in it to remain at least 150% collateralized.

    return await cdp.drawDai(10000000000000000000, DAI.wei);
    // or equivalently
    return await cdp.drawDai(100, DAI);


## **wipeDai**


- **Params:** amount to repay (in Dai, as string)
- **Returns:** promise (resolves to `[transactionObject](https://makerdao.com/documentation/#transactions)` once mined)

`cdp.wipeDai(dai)` sends Dai back to the CDP in order to repay some (or all) of its outstanding debt.
*Note: CDPs accumulate MKR governance debt over their lifetime. This must be paid when wiping dai debt, and thus MKR must be acquired before calling this method.*


    return await cdp.wipeDai(10000000000000000000, DAI.wei);
    // or equivalently
    return await cdp.wipeDai(100, DAI);



## **freePeth**


- **Params:** amount of Peth collateral to free from the CDP, in units defined by the price service.
- **Returns:** promise (resolves to `[transactionObject](https://makerdao.com/documentation/#transactions)` once mined)

`cdp.freePeth(peth)` withdraws the specified amount of PETH and returns it to the owner's address. As such, the contract will only allow you to free PETH that's locked in excess of 150% of the CDP's outstanding debt.


    return await cdp.freePeth(100, PETH);
    // or equivalently
    return await cdp.freePeth(10000000000000000000, PETH.wei);



## **give**

**Params:** Ethereum address (string)

- **Returns:** promise (resolves to `[transactionObject](https://makerdao.com/documentation/#transactions)` once mined)

`cdp.give(address)` transfers ownership of the CDP from the current owner to the address you provide as an argument.


    return await cdp.give('0x046ce6b8ecb159645d3a605051ee37ba93b6efcc');



## **shut**

**Params:** none

- **Returns:** promise (resolves to `[transactionObject](https://makerdao.com/documentation/#transactions)` once mined)

`cdp.shut()` wipes all remaining dai, frees all remaining collateral, and deletes the CDP. This will fail if the caller does not have enough DAI and MKR to wipe all debt.


    return await cdp.shut();



## **bite**

**Params:** none

- **Returns:** promise (resolves to `[transactionObject](https://makerdao.com/documentation/#transactions)` once mined)

`cdp.bite()` will initiate the liquidation process of an undercollateralized CDP


    return await cdp.bite();


# **Price Service**

Retrieve the PriceService through `Maker.service('price')`.
The PriceService exposes the collateral and governance tokens' price information that is reported by the oracles in the Maker system.


    const price = maker.service('price');


## **getEthPrice**

Get the current USD price of ETH, as a `USD_ETH` [price unit](https://makerdao.com/documentation/#units).


    const ethPrice = await price.getEthPrice();


## **getMkrPrice**

Get the current USD price of the governance token MKR, as a `USD_MKR` [price unit](https://makerdao.com/documentation/#units).


    const mkrPrice = await price.getMkrPrice();


## **getPethPrice**

Get the current USD price of PETH (pooled ethereum), as a `USD_PETH` [price unit](https://makerdao.com/documentation/#units).


    await pethPrice = price.getPethPrice();



## **setEthPrice**

Set the current USD price of ETH.
This requires the necessary permissions and will only be useful in a testing environment.


    await price.setEthPrice(475);



## **setMkrPrice**

Set the current USD price of the governance token MKR.
This requires the necessary permissions and will only be useful in a testing environment.

    await price.setMkrPrice(950.00);


## **getWethToPethRatio**

Returns the current W-ETH to PETH ratio.


    await price.getWethToPethRatio();
# **ETH CDP Service**

Retrieve the ETH CDP Service through Maker.service('cdp').
The ETH CDP Service exposes the risk parameter information for the Ether CDP type (in single-collateral Dai, this is the only CDP Type)


    const ethCdp = maker.service('cdp');


## **getLiquidationRatio**

**Params:** none

- **Returns:** promise (resolves to liquidation ratio)

`getLiquidationRatio()` returns a decimal representation of the liquidation ratio, e.g. 1.5


    const ratio = await ethCdp.getLiquidationRatio();
## **getLiquidationPenalty**

**Params:** none

- **Returns:** promise (resolves to liquidation penalty)

`getLiquidationPenalty()` returns a decimal representation of the liquidation penalty, e.g. 0.13


    const penalty = await ethCdp.getLiquidationPenalty();
## **getAnnualGovernanceFee**

**Params:** none

- **Returns:** promise (resolves to yearly governance fee)

`getAnnualGovernanceFee()` returns a decimal representation of the annual governance fee, e.g. 0.005.


    const fee = await ethCdp.getAnnualGovernanceFee();

*Note: this is often referred to as the* `*Stability Fee*`*, even though technically the* `*Stability Fee*` *is the fee that is paid in Dai, and the* `*Governance Fee*` *is the fee that is paid in MKR. But since fees are only paid in MKR in Single-Collateral Dai, and only paid in Dai in Multi-Collateral Dai, the fee in Single-Collateral Dai is often referred to as the* `*Stability Fee*` *to be consistent with the term that will be used in Multi-Collateral Dai and to avoid unduly confusing regular users.*



# **Exchange Service**

Retrieve the OasisExchangeService (or alternative implementation) through `Maker.service('exchange')`.
The exchange service allows to buy and sell DAI, MKR, and other tokens. The default OasisExchangeService implementation uses the OasisDEX OTC market for this.


    const exchange = maker.service('exchange');



## **sellDai**

Sell a set amount of DAI and receive another token in return.

- **Parameters**
  - `daiAmount` - Amount of DAI to sell.
  - `tokenSymbol` - Token to receive in return.
  - `minFillAmount` - Minimum amount to receive in return.
- **Returns:** promise (resolves to [OasisOrder](https://makerdao.com/documentation/#oasisorder) once mined)


    // Sell 100.00 DAI for 0.30 WETH or more.
    const sellOrder = await exchange.sellDai('100.0', 'WETH', '0.30');
## **buyDai**

Buy a set amount of DAI and give another token in return.

- **Parameters**
  - `daiAmount` - Amount of DAI to buy.
  - `tokenSymbol` - Token to give in return.
  - `minFillAmount` - Maximum amount to give in return.
- **Returns:** promise (resolves to [OasisOrder](https://makerdao.com/documentation/#oasisorder) once mined)


    // Buy 100.00 DAI for 0.30 WETH or less.
    const buyOrder = await exchange.buyDai('100.0', 'WETH', '0.35');
## **OasisOrder**

`OasisOrders` have a few methods: * `fillAmount`: amount of token received in exchange * `fees()`: amount of ether spent on gas * `created()`: timestamp of when transaction was mined


    const buyOrder = await exchange.buyDai('100.0', 'WETH', '0.35');
    const fillAmount = buyOrder.fillAmount();
    const gasPaid = buyOrder.fees();
    const created = buyOrder.created();
# **System Status**

To access system status information, retrieve the ETH CDP Service through Maker.service('cdp').


    const ethCdp = maker.service('cdp');
## **getSystemCollateralization**

**Params:** none

- **Returns:** promise (resolves to system collateralization ratio)

`getSystemCollateralization()` returns the collateralization ratio for the entire system, e.g. 2.75


    const systemRatio = await ethCdp.getSystemCollateralization();
## **getTargetPrice**

**Params:** none

- **Returns:** promise (resolves to target price)

`getTargetPrice()` returns the target price of Dai in USD, that is, the value to which Dai is soft-pegged, e.g. 1.0

    const targetPrice = await ethCdp.getTargetPrice();
# **Events**

The event pipeline allows developers to easily create real-time applications by letting them listen for important state changes and lifecycle events.
**Wildcards**
An event name passed to any event emitter method can contain a wildcard (the `*`character). A wildcard may appear as `foo/*`, `foo/bar/*`, or simply `*`.
`*` matches one sub-level.
e.g. `price/*` will trigger on both `price/USD_ETH` and `price/MKR_USD` but not `price/MKR_USD/foo`.
`**` matches all sub-levels.
e.g. `price/**` will trigger on `price/USD_ETH`, `price/MKR_USD`, and `price/MKR_USD/foo`.
**Event Object**

Triggered events will receive the object shown on the right.

- `<event_type>` - the name of the event
- `<event_payload>` - the new state data sent with the event
- `<event_sequence_number>` - a sequentially increasing index
- `<latest_block_when_emitted>` - the current block at the time of the emit


    {
        type: <event_type>,
        payload: <event_payload>, /* if applicable */
        index: <event_sequence_number>,
        block: <latest_block_when_emitted>
    }


## **Maker Object**

`**Price**`


| Event Name        | Payload   |
| ----------------- | --------- |
| `price/ETH_USD`   | { price } |
| `price/MKR_USD`   | { price } |
| `price/WETH_PETH` | { ratio } |

    maker.on('price/ETH_USD', eventObj => {
        const { price } = eventObj.payload;
        console.log('ETH price changed to', price);
    })


`**Web3**`


| Event Name             | Payload                     |
| ---------------------- | --------------------------- |
| `web3/INITIALIZED`     | { provider: { type, url } } |
| `web3/CONNECTED`       | { api, network, node }      |
| `web3/AUTHENTICATED`   | { account }                 |
| `web3/DEAUTHENTICATED` | { }                         |
| `web3/DISCONNECTED`    | { }                         |

    maker.on('web3/AUTHENTICATED', eventObj => {
        const { account } = eventObj.payload;
        console.log('web3 authenticated with account', account);
    })
## **CDP Object**
| Event Name   | Payload      |
| ------------ | ------------ |
| `COLLATERAL` | { USD, ETH } |
| `DEBT`       | { dai }      |

    cdp.on('DEBT', eventObj => {
        const { dai } = eventObj.payload;
        console.log('Your cdp now has a dai debt of', dai);
    })
# **Transactions**

**TransactionObject Lifecycle Hooks**

The `transactionManager` service is used to access and monitor `TransactionObjects` which can be used to track a transaction's status as it propagates through the blockchain. `TransactionObjects` have methods which trigger callbacks in the event of a transaction's status being `pending`, `mined`, `confirmed` or `error`.


    const txMgr = maker.service('transactionManager');
    // instance of transactionManager
    const open = maker.service('cdp').openCdp();
    // open is a transactionObject when promise resolves


**Listen**

Accessing these status callbacks is done through the `listen` function from the `transactionManager` service. The listen function takes a promise and a callback object with the transaction status as a key.
One caveat is that the `confirmed` event will not fire unless the `transactionManager` confirms the promise. This `confirm()` function waits on a number of blocks after the transaction has been mined, of which the default is 5, to resolve. To change this, the `confirmedBlockCount` attribute in the [options](https://makerdao.com/documentation/#options) object can be modified.



    txMgr.listen(open, {
      pending: tx => {
        // do something when tx is pending
      },
      mined: tx => {
        // do something when tx is mined
      },
      confirmed: tx => {
        // do something when tx is confirmed       
      },
      error: tx => {
        // do someting when tx fails
      }
    });
    
    await txMgr.confirm(open); // confirmed will fire after 5 blocks if default


**Transaction Metadata**

There are functions such as `lockEth()` which are composed of several internal transactions. These can be more accurately tracked by accessing the `tx.metadata`in the callback which contains both the `contract` and the `method` the internal transactions were created from.


    const lock = cdp.lockEth(1);
    txMgr.listen(lock, {
      pending: tx => {
        const {contract, method} = tx.metadata;
        if(contract === 'WETH' && method === 'deposit') {
          console.log(tx.hash); // print hash for WETH.deposit
        }
      }
    })

**TransactionObject Methods**

A `TransactionObject` also has a few methods to provide further details on the transaction:

- `hash` : transaction hash
- `fees()` : amount of ether spent on gas
- `timeStamp()` : timestamp of when transaction was mined
- `timeStampSubmitted()` : timestamp of when transaction was submitted to the network


# **Using Multiple Accounts**

The library supports the use of multiple accounts (i.e. private keys) within a single Maker instance. Accounts can be specified in the constructor options or with the `addAccount` method.
Call `useAccount` to switch to using an account by its name, or `useAccountWithAddress` to switch to using an account by its address, and subsequent calls will use that account as the transaction signer.

When the Maker instance is first created, it will use the account named `default` if it exists, or the first account in the list otherwise.


    const maker = await Maker.create({
      url: 'http://localhost:2000',
      accounts: {
        other: {type: privateKey, key: someOtherKey},
        default: {type: privateKey, key: myKey}
      }
    });
    
    await maker.authenticate();
    await maker.addAccount('yetAnother', {type: privateKey, key: thirdKey});
    
    const cdp1 = await maker.openCdp(); // owned by "default"
    
    maker.useAccount('other');
    const cdp2 = await maker.openCdp(); // owned by "other"
    
    maker.useAccount('yetAnother');
    const cdp3 = await maker.openCdp(); // owned by "yetAnother"
    
    await maker.addAccount({type: privateKey, key: fourthAccount.key}); // the name argument is optional
    maker.useAccountWithAddress(fourthAccount.address)
    const cdp4 = await maker.openCdp(); //owned by the fourth account



## **Account types**

In addition to the `privateKey` account type, there are two other built-in types:

- `provider`: Get the first account from the provider (e.g. the value from `getAccounts`).
- `browser`: Get the first account from the provider in the browser (e.g. MetaMask), even if the Maker instance is configured to use a different provider.


    const maker = await Maker.create({
      url: 'http://localhost:2000',
      accounts: {
        // this will be the first account from the provider at
        // localhost:2000
        first: {type: 'provider'},
    
        // this will be the current account in MetaMask, and it
        // will send its transactions through MetaMask
        second: {type: 'browser'},
    
        // this account will send its transactions through the
        // provider at localhost:2000
        third: {type: 'privateKey', key: myPrivateKey}
      }
    })


## **Plugins**

Plugins can add additional account types. There are currently two such plugins for hardware wallet support:

- [dai-plugin-trezor-web](https://github.com/makerdao/dai-plugin-trezor-web)
- [dai-plugin-ledger-web](https://github.com/makerdao/dai-plugin-ledger-web)


    import TrezorPlugin from '@makerdao/dai-plugin-trezor-web';
    import LedgerPlugin from '@makerdao/dai-plugin-ledger-web';
    
    const maker = await Maker.create({
      plugins: [
        TrezorPlugin,
        LedgerPlugin,
      ],
      accounts: {
        // default derivation path is "44'/60'/0'/0/0"
        myTrezor: {type: 'trezor', path: derivationPath1},
        myLedger: {type: 'ledger', path: derivationPath2}
      }
    });


## **Demo**

Install the [multiple accounts demo app](https://github.com/makerdao/integration-examples/tree/master/accounts) to see this functionality in action.



# **Using DSProxy**

The `DSProxyService` includes all the functionality necessary for interacting with both types of proxies found in Maker products: *profile proxies* and *forwarding proxies*.
Forwarding proxies are simple contracts that aggregate function calls in the body of a single method. These are used in the [CDP Portal](https://github.com/makerdao/sai-proxy) and [Oasis Direct](https://github.com/makerdao/oasis-direct-proxy) in order to allow users to execute multiple transactions atomically, which is both safer and more user-friendly than implementing several steps as discrete transactions.


    // Forwarding proxy
    
    function lockAndDraw(address tub_, bytes32 cup, uint wad) public payable {
      lock(tub_, cup);
      draw(tub_, cup, wad);
    }

Forwarding proxies are meant to be as simple as possible, so they lack some features that could be important if they are to be used as interfaces for more complex smart contract logic. This problem can be solved by using profile proxies (i.e., copies of [DSProxy](https://github.com/dapphub/ds-proxy)) to execute the functionality defined in the forwarding proxies.
The first time an account is used to interact with any Maker application, the user will be prompted to deploy a profile proxy. This copy of DSProxy can be used in any product, including dai.js, by way of a universal [proxy registry](https://github.com/makerdao/proxy-registry/tree/master). Then, the calldata from any function in the forwarding proxy can be passed to DSProxy's `execute()`method, which runs the provided code in the context of the profile proxy.


    // Calling the forwarding proxy with dai.js
    
    function lockAndDraw(tubContractAddress, cdpId, daiAmount, ethAmount) {
      const saiProxy = maker.service('smartContract').getContractByName('SAI_PROXY');
    
      return saiProxy.lockAndDraw(
        tubContractAddress,
        cdpId,
        daiAmount,
        {
          value: ethAmount,
          dsProxy: true
        }
      );
    }
    

This makes it possible for users' token allowances to persist from one Maker application to another, and it allows users to [recover any funds](https://proxy-recover-funds.surge.sh/) mistakenly sent to the proxy's address.
Many of the functions in `DSProxyService` will only be relevant to `power users`. All that is strictly required to automatically generate a function's calldata and find the correct profile proxy is the inclusion of `{ dsProxy: true }` in the options object for any transaction — provided the user has already deployed a profile proxy. If that's not certain, it may also be necessary to query the registry to determine if a user already owns a proxy, and to `build` one if they do not.


## **currentProxy**


- **Params:** None
- **Returns:** promise (resolves to address **or** `null`)

If the `currentAccount` (according the `Web3Service`) has already deployed a DSProxy, `currentProxy()` returns its address. If not, it returns `null`. It will update automatically in the event that the active account is changed.
This function should be used to check whether a user has a proxy before attempting to build one.


    asnyc function getProxy() {
      return maker.service('proxy').currentProxy();
    }
## **build**


- **Params:** None
- **Returns:** `TransactionObject`

`build` will deploy a copy of DSProxy owned by the current account. **This transaction will revert if the current account already owns a profile proxy.**
By default, `build()` returns after the transaction is mined.


    async function buildProxy() {
      const proxyService = maker.service('proxy');
      if (!proxyService.currentProxy()) {
        return await proxyService.build();
      }
    }


## **execute**


    

This function is designed to be called by the `TransactionManager`, not accessed directly. As long as the target contract is defined by the `SmartContractService`, the inclusion of the `dsProxy` key in the options for any transaction will automatically forward the transaction to the `DSProxyService`.

The value of `dsProxy` can either be `true` or an explicitly provided DSProxy address.


    function lockAndDraw(tubContractAddress, cdpId, daiAmount, ethAmount) {
      const saiProxy = maker.service('smartContract').getContractByName('SAI_PROXY');
    
      return saiProxy.lockAndDraw(
        tubContractAddress,
        cdpId,
        daiAmount,
        {
          value: ethAmount,
          dsProxy: true
        }
      );
    }


## **getProxyAddress**

**Params:** Address (optional)

- **Returns:** promise (resolves to contract address)

`getProxyAddress` will query the proxy registry for the profile proxy address associated with a given account. If no address is provided as a parameter, the function will return the address of the proxy owned by the `currentAccount`.


    const proxy = await maker.service('proxy').getProxyAddress('0x...');
## **getOwner**

**Params:** Address

- **Returns:** promise (resolves to address)

`getOwner` will query the proxy registry for the owner of a provided instance of DSProxy.


    const owner = await maker.service('proxy').getOwner('0x...');



## **setOwner**


- **Params:** Address of new owner, DSProxy address (optional)
- **Returns:** `TransactionObject`

`setOwner` can be used to give a profile proxy to a new owner. The address of the recipient account must be specified, but the DSProxy address will default to `currentProxy` if the second parameter is excluded.


    async function giveProxy(newOwner, proxyAddress) {
      return await maker.service('proxy').setOwner(newOwner, proxyAddress);
    }
# **Developing**
1. `git clone https://github.com/makerdao/makerdao-integration-poc`
2. `npm install`
## **Running the tests**

The test suite is configured to run on a Ganache test chain.
If you run the tests with `npm test`, it will start a test chain and deploy all the smart contracts for the Dai stablecoin system. This takes a few minutes.
To keep the test chain running and re-run the tests when changes are made to the code, use the command `npm run test:watch`.
If you want to deploy the contracts to a test chain without running the test suite, use `npm run test:net`.

## **Inspect contract state**

Start the dev server using `npm start`, then open [http://localhost:9000](http://localhost:9000/).

## **Commands**
- `npm start` - start the dev server
- `npm run build:backend` - create backend build in `dist` folder
- `npm run build:frontend` - create frontend build in `dist` folder
- `npm run lint` - run an ESLint check
- `npm run coverage` - run code coverage and generate report in the `coverage`folder
- `npm test` - run all tests
- `npm run test:watch` - run all tests in watch mode
- `npm run test:net` - launch a Ganache test chain and deploy MakerDAO's contracts on it


# **Adding a New Service**

*Note: This section is only for advanced users that are willing to modify the dai.js source code. In the future, you will be able to add custom services without needing to modify the source code.*
You can take advantage of the pluggable architecture of this library by choosing different implementations for services, and/or adding new service roles altogether. A service is just a Javascript class that inherits from either `PublicService`, `PrivateService`, or `LocalService`, and contains public method(s). It can depend on other services through a built-in dependency injection framework, and can also be configured through the Maker config file / config options.


## **Steps to add a new service**

Here are the steps to add a new service called `ExampleService` to MakerJS:

- (1) In the `src` directory, create an `ExampleService.js` file in one of the subdirectories.
- (2) In `src/config/DefaultServiceProvider.js`, import `ExampleService.js` and add it to the `_services` array
- (3) Create a class called `ExampleService` in `ExampleService.js`


    //example code in ExampleService.js for steps 3-6
    import PublicService from '../core/PublicService';
    
    export default class ExampleService extends PublicService {
        constructor (name='example') {
            super(name, ['log']);
        }
    
        test(){
            this.get('log').info('test');
        }


- (4) The service must `extend` one of:
  - `PrivateService` - requires both a network connection and authentication
  - `PublicService` - requires just a network connection
  - `LocalService` - requires neither
- See the `Service Lifecycle` section below for more info
- (5) In the constructor, call the parent class's constructor with the following arguments:
  - The name of the service. This is how the service can be referenced by other services
  - An array of the names of services to depend on
- (6) Add the necessary public methods


    //example code in ExampleService.spec.js for step 8
    import Maker from '../../src/index';
    
    //step 8: a new service role ('example') is used
    test('test 1', async () => {
      const maker = await Maker.create('http', {example: "ExampleService"});
      const exampleService = customMaker.service('example');
      exampleService.test(); //logs "test"
    });
    
    //step 8: a custom service replaces a default service (Web3)
    test('test 2', async () => {
      const maker = await Maker.create('http', {web3: "MyCustomWeb3Service"});
      const mycustomWeb3Service = maker.service('web3');
    });


- (7) If your service will be used to replace a default service (the full list of default service roles can be found in `src/config/ConfigFactory`), then skip this step. Otherwise, you'll need to add your new service role (e.g. `"example"`) to the `ServiceRoles` array in `src/config/ConfigFactory`.
- (8) Create a corresponding `ExampleService.spec.js` file in the `test` directory. Write a test in the test file that creates a Maker object using your service.
- (9) (Optional) Implement the relevant service lifecycle functions (`initialize()`, `connect()`, and `authenticate()`). See the `Service Lifecycle` section below for more info


    //step 10: in ExampleService.spec.js
    const maker = await Maker.create('http', {
        example: ["ExampleService", {
        exampleSetting: "this is a configuration setting"
      }]
    });
    
    //step 10: accessing configuration settings in ExampleService.js
    initialize(settings) {
      if(settings.exampleSetting){
        this.get('log').info(settings.exampleSetting);
      }
    }


- (10) (Optional) Allow for configuration. Service-specific settings can be passed into a service by the Maker config file or config options. These service-specific settings can then be accessed from inside a service as the parameter passed into the `initialize` function (see the `Service Lifecycle` section below)


## **Service Lifecycle**

The three kinds of services mentioned in step 4 above follow the following state machine diagrams in the picture below.


    //example initialize() function in ExampleService.js
      initialize(settings) {
        this.get('log').info('ExampleService is initializing...');
        this._setSettings(settings);
      }

To specify what initializing, connecting and authenticating entails, implement the `initialize()`, `connect()`, and `authenticate()` functions in the service itself. This will be called while the service's manager brings the service to the corresponding state.



![alt text](https://makerdao.com/documentation/images/statemachine.png)


A service will not finish initializing/connecting/authenticating until all of its dependent services have completed the same state (if applicable - for example a `LocalService` is considered authenticated/connected in addition to initialized, if it has finished initializing). The example code here shows how to wait for the service to be in a certain state.


    const maker = await Maker.create('http', {example: "ExampleService"});
    const exampleService = customMaker.service('example');
    
    //wait for example service and its dependencies to initialize
    await exampleService.manager().initialize();
    
    //wait for example service and its dependencies to connect
    await exampleService.manager().connect();
    
    //wait for example service and its dependencies to authenticate
    await exampleService.manager().authenticate();
    
    //can also use callback syntax
    exampleService.manager().onConnected(()=>{
        /*executed after connected*/
    });
    
    //wait for all services used by the maker object to authenticate
    maker.authenticate();


## **Adding Custom Events**

One way to add an event is to “register” a function that gets called on each new block, using the event service's `registerPollEvents()` function. For example, here is some code from the price service. `this.getEthPrice()` will be called on each new block, and if the state has changed from the last call, a `price/ETH_USD` event will be emitted with the payload { price: [new_price] }.


    //in PriceService.js
    this.get('event').registerPollEvents({
          'price/ETH_USD': {
            price: () => this.getEthPrice()
          }
        });


Another way to an add an event is to manually emit an event using the event service's `emit` function. For example, when the Web3Service initializes, it emits an event that contains info about the provider.


    //in Web3Service.js
    this.get('event').emit('web3/INITIALIZED', {
      provider: { ...settings.provider }
    });

Note that calling `registerPollEvents` and `emit()` directly on the event service like in the previous two examples will register events on the "default" event emitter instance. However, you can create a new event emitter instance for your new service. For example, the CDP object defines it's own event emitter, as can be seen here, by calling the event service's `buildEmitter()` function.


    //in the constructor in the Cdp.js
    this._emitterInstance = this._cdpService.get('event').buildEmitter();
    this.on = this._emitterInstance.on;
    this._emitterInstance.registerPollEvents({
      COLLATERAL: {
        USD: () => this.getCollateralValueInUSD(),
        ETH: () => this.getCollateralValueInEth()
      },
      DEBT: {
        dai: () => this.getDebtValueInDai()
      }
    });



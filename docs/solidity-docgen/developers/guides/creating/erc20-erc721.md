# ERC20/ERC721

This guide aims to teach developers how to structure a swap for ERC20 and ERC721 tokens within the Swaplace project. We'll focus on creating swaps using the`Swaplace` smart contract, which facilitates token swaps. The process involves defining the swap parameters, encoding configurations, and finally creating the swap.

> Check the testsuit for this case [here](https://github.com/blockful-io/swaplace-contracts/blob/026d8cc3bc871936a2737e6e45ae1afb290dd05d/test/TestSwaplace.test.ts#L221)

## Understanding the Swap Parameters

A swap in this context refers to an exchange of tokens between parties. For ERC721 tokens, these could represent unique digital items, while ERC20 tokens are fungible tokens representing a specific value. A swap can involve either a single token (1-1), multiple tokens of the same type (N-N), or a combination thereof.

### Key Parameters

* **Owner**: The address initiating the swap.
* **Allowed**: The address permitted to accept the swap. Setting this to the zero address allows anyone to accept.
* **Expiry**: Timestamp indicating when the swap offer expires.
* **Recipient**: Determines who receives any ETH involved in the swap. `0` means the acceptor, values `1` through `255` indicate the swap owner.
* **Value**: Amount of ETH involved in the swap, with a maximum precision of 6 decimals.
* **Assets**: Array of assets being offered in the swap.
* **Asking**: Array of assets requested in exchange.

## Step-by-Step Implementation

### 1. Define Swap Parameters

First, define the parameters for your swap. This includes specifying the addresses of the tokens involved, the amounts or IDs (for ERC721), and the configuration details like expiry and recipient.

* Addresses of the tokens
  * biddingAddr
  * askingAddr
* Amounts or IDs of the tokens
  * biddingAmountOrId
  * askingAmountOrId
* Configuration object
  * config

These parameters are crucial for specifying what assets are being swapped and under what conditions.

<pre class="language-typescript"><code class="lang-typescript">//ERC20
const bidingAddr = [MockERC721.address]; // Address of the token being offered
const bidingAmountOrId = [1]; // ID of the ERC721 token being offered

const askingAddr = [MockERC721.address]; // Address of the token requested
const askingAmountOrId = [4]; // ID of the ERC721 token requested
<strong>
</strong><strong>//ERC721
</strong><strong>const bidingAddr = [MockERC20.address]; // Address of the token being offered
</strong>const bidingAmountOrId = [500]; // Amount of the ERC20 token being offered

const askingAddr = [MockERC20.address]; // Address of the token requested
const askingAmountOrId = [200]; // Amount of the ERC20 token requested
</code></pre>

### 2: Encode Swap Configuration

Use the `encodeConfig` function to encode the allowed address, expiry timestamp, recipient type, and value into a single uint256 value. This simplifies passing these parameters to the swap creation function.

* Address of the recipient
  * Zero Address or a valid Address
* currentTimestamp
  * a timestamp indicating when the swap becomes valid

```typescript
const currentTimestamp = (await blocktimestamp()) + 1000000; // Future timestamp for expiry
const config = await Swaplace.encodeConfig(zeroAddress, currentTimestamp, 0, 0);
```

### 3: Compose the Swap

The `composeSwap` function takes the owner's address, the encoded configuration, and arrays of token addresses and amounts/IDs for both sides of the swap. This function constructs a structured representation of the swap that can be passed to the smart contract.

```javascript
const swap = await composeSwap(
  owner.address,
  config,
  bidingAddr,
  bidingAmountOrId,
  askingAddr,
  askingAmountOrId,
);
```

### Step 4: Create the Swap on the Blockchain

Finally, create the swap by calling the `createSwap` function on the `Swaplace` contract, passing in the composed swap structure. Ensure the transaction is sent from the owner's address.

```typescript
await expect(await Swaplace.connect(owner).createSwap(swap))
  .to.emit(Swaplace, "SwapCreated")
  .withArgs(await Swaplace.totalSwaps(), owner.address, zeroAddress);
```

> You can check the full code [here](https://github.com/blockful-io/swaplace-contracts/blob/026d8cc3bc871936a2737e6e45ae1afb290dd05d/contracts/Swaplace.sol)

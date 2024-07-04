# Native Ether

This guide provides a step-by-step approach for developers looking to implement the swap mechanism for `native Ethers`within the Swaplace project.  The process involves defining the swap parameters, encoding configurations, and finally creating the swap.

> Check the testsuit for this case [here](https://github.com/blockful-io/swaplace-contracts/blob/026d8cc3bc871936a2737e6e45ae1afb290dd05d/test/TestSwaplace.test.ts#L514)

## Understanding Swap Parameters with Native Ethers

When dealing with native Ether in swaps, alongside ERC20, ERC721 and ERC1155 tokens, the key parameters remain similar but with added attention to the Ether value being transferred:

* **Owner**: The Ethereum address initiating the swap.
* **Allowed**: The address authorized to accept the swap. A zero address implies anyone can accept.
* **Expiry**: A future timestamp indicating when the swap offer expires.
* **Recipient**: Specifies who receives the Ether involved in the swap. `0` denotes the acceptor, while values `1` through `255` refer to the swap owner.
* **Value**: The amount of Ether involved in the swap, with a precision of up to 6 decimals.
* **Assets**: An array detailing the assets being offered in the swap, including token addresses, IDs, and amounts.
* **Asking**: An array specifying what assets are requested in exchange.

{% hint style="info" %}
**NOTE**

* If the recipient is set to '0', then the swap creator sends Ether to the one accepting the swap.
* If the recipient is set to '1', then the acceptee send Ether to the one who created the swap.
{% endhint %}

## Step-by-Step Implementation

### 1.Define Swap Parameters

Define the parameters for your swap, including the addresses of the tokens involved, their amounts, and configuration details like expiry and recipient.

```typescript
//ERC20
const bidingAddr = [MockERC721.address]; // Address of the token being offered
const bidingAmountOrId = [1]; // ID of the ERC721 token being offered

const askingAddr = [MockERC721.address]; // Address of the token requested
const askingAmountOrId = [4]; // ID of the ERC721 token requested

//ERC721
const bidingAddr = [MockERC20.address]; // Address of the token being offered
const bidingAmountOrId = [500]; // Amount of the ERC20 token being offered

const askingAddr = [MockERC20.address]; // Address of the token requested
const askingAmountOrId = [200]; // Amount of the ERC20 token requested
```

### 2. Calculate Ether Value

Determine the amount of Ether to send with the swap. Use utility functions like `ethers.utils.parseEther` to convert Ether units to wei, the smallest unit of Ether.

```typescript
const valueToSend: BigNumber = ethers.utils.parseEther("0.5"); // Convert 0.5 Ether to wei
```

### 3. Encode configuration

Encode the allowed address, expiry timestamp, recipient type, and Ether value into a single uint256 value using `encodeConfig`.

{% hint style="info" %}
**NOTE**&#x20;

The Ether value must be divided by 1e12 because the minimum accepted must carry 6 decimals to fit correctly in a single storage slot
{% endhint %}

```typescript
const currentTimestamp = (await blocktimestamp()) + 1000000; // Future timestamp for expiry
const config = await Swaplace.encodeConfig(zeroAddress, currentTimestamp, 0, valueToSend.div(1e12));
```

### 4. Compose the Swap

Compose the swap structure by calling the function called `omposeSwap`, passing in the owner's address, encoded configuration, and arrays detailing the bid and ask components.

```typescript
const swap = await composeSwap(owner.address, config, bidingAddr, bidingAmountOrId, askingAddr, askingAmountOrId);
```

### 5. Create the Swap with Ether

Finally, create the swap by calling the `createSwap` function on the `Swaplace` contract, passing in the composed swap structure and the Ether value as part of the transaction options.

```typescript
await expect(
  await Swaplace.connect(owner).createSwap(swap, { value: valueToSend }),
)
.to.emit(Swaplace, "SwapCreated")
.withArgs(await Swaplace.totalSwaps(), owner.address, zeroAddress);
```

> You can check the full code [here](https://github.com/blockful-io/swaplace-contracts/blob/026d8cc3bc871936a2737e6e45ae1afb290dd05d/contracts/Swaplace.sol)

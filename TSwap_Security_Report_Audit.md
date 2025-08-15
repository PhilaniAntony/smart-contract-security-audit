---
title: "TSwap Smart Contract Security Audit Report"
author: "Philani A Dlamini"
date: "August 2025"
geometry: margin=1in
fontsize: 11pt
toc: true
toc-depth: 2
numbersections: true
colorlinks: true
linkcolor: blue
titlepage: true
titlepage-color: "E0E0E0"
titlepage-text-color: "000000"
titlepage-rule-color: "000000"
titlepage-rule-height: 2
---

# TSwap Smart Contract Security Audit

![TSwap Protocol Architecture](/images/tswap.png)

## Vulnerability Summary

| Severity Level   | Count |
| ---------------- | ----- |
| **High**         | 3     |
| **Low**          | 2     |
| **Informational**| 6     |

---

## Table of Contents

- [TSwap Smart Contract Security Audit](#tswap-smart-contract-security-audit)
  - [Vulnerability Summary](#vulnerability-summary)
  - [Table of Contents](#table-of-contents)
  - [High Severity Findings](#high-severity-findings)
    - [\[H-1\] Reentrancy Risk in `PoolFactory::createPool`](#h-1-reentrancy-risk-in-poolfactorycreatepool)
    - [\[H-2\] Missing Deadline Check in deposit `TSwapPool::deposit`](#h-2-missing-deadline-check-in-deposit-tswappooldeposit)
    - [\[H-3\] Incorrect fee calculation in `TSwap::getInputAmountBasedOnOutput`](#h-3-incorrect-fee-calculation-in-tswapgetinputamountbasedonoutput)
  - [Low Severity Findings](#low-severity-findings)
    - [\[L-1\] `TSwap::_addLiquidityMintAndTransfer` function emits `TSwap::LiquidityAdded` with parameters out of order](#l-1-tswap_addliquiditymintandtransfer-function-emits-tswapliquidityadded-with-parameters-out-of-order)
    - [\[L-2\] Default value returned by  `TSwapFactory::swapExactInput` function is not updated resulting in an incorrect return value](#l-2-default-value-returned-by--tswapfactoryswapexactinput-function-is-not-updated-resulting-in-an-incorrect-return-value)
  - [Informationals](#informationals)
    - [\[I-1\] `PoolFactory::PoolFactory__PoolDoesNotExist`is not used and should be removed](#i-1-poolfactorypoolfactory__pooldoesnotexistis-not-used-and-should-be-removed)
    - [\[I-2\] Lacking 0 address checks](#i-2-lacking-0-address-checks)
    - [\[I-3\] `PoolFactory::createPool` should use `symbol()` instead of `name()`](#i-3-poolfactorycreatepool-should-use-symbol-instead-of-name)
    - [\[I-4\] `PoolFactory::PoolCreated` events is missing an indexed field](#i-4-poolfactorypoolcreated-events-is-missing-an-indexed-field)
    - [\[I-5\] `TSwapFactory::swapExactInput` function is marked as public and not used internally, it should be updated to external](#i-5-tswapfactoryswapexactinput-function-is-marked-as-public-and-not-used-internally-it-should-be-updated-to-external)
    - [\[I-6\] `TSwapFactory::revertIfZero` modifier has a strict zero check](#i-6-tswapfactoryrevertifzero-modifier-has-a-strict-zero-check)

---

## High Severity Findings

### [H-1] Reentrancy Risk in `PoolFactory::createPool`

**Description:**  
The `createPool` function updates contract state **after** making external calls to retrieve `name()` and `symbol()` from the token contract. This opens the contract to potential **reentrancy attacks**.

**Impact:**  

- **Loss of funds:** A malicious token could exploit this to drain the pool.

**Proof of Concept:** 

```diff
function createPool(address tokenAddress) external returns (address) {
    if (s_pools[tokenAddress] != address(0)) {
        revert PoolFactory__PoolAlreadyExists(tokenAddress);
    }
+   TSwapPool tPool = new TSwapPool(
+       tokenAddress, i_wethToken, liquidityTokenName, liquidityTokenSymbol
+   );
+   s_pools[tokenAddress] = address(tPool);
+   s_tokens[address(tPool)] = tokenAddress;
-   string memory liquidityTokenName = string.concat("T-Swap ", IERC20(tokenAddress).name());
-   string memory liquidityTokenSymbol = string.concat("ts", IERC20(tokenAddress).name());
-   TSwapPool tPool = new TSwapPool(
-       tokenAddress, i_wethToken, liquidityTokenName, liquidityTokenSymbol
-   );
-   s_pools[tokenAddress] = address(tPool);
-   s_tokens[address(tPool)] = tokenAddress;
+   string memory liquidityTokenName = string.concat("T-Swap ", IERC20(tokenAddress).name());
+   string memory liquidityTokenSymbol = string.concat("ts", IERC20(tokenAddress).name());

    emit PoolCreated(tokenAddress, address(tPool));
    return address(tPool);
}
```

**Recommended Mitigation:** 

- **Adopt the Checks-Effects-Interactions (CEI) pattern:** update state before making external calls.

### [H-2] Missing Deadline Check in deposit `TSwapPool::deposit` 

**Description:** 
The `deposit` function takes a `deadline` parameter but never checks it. This allows liquidity deposits to execute after the intended time.

**Impact:** 

- **Unexpected execution:** Deposits could be executed under unfavorable market conditions.

**Recommended Mitigation:** 

Add a revert condition if the deadline has passed.

```diff
function deposit(uint256 wethToDeposit, uint256 minimumLiquidityTokensToMint,
     uint256 maximumPoolTokensToDeposit, uint64 deadline
) external revertIfZero(wethToDeposit) returns (uint256 liquidityTokensToMint) {
+   if (uint64(block.timestamp) > deadline) revert();
    ...
}
```

### [H-3] Incorrect fee calculation in `TSwap::getInputAmountBasedOnOutput`

**Description:** 
The fee calculation uses 10000 instead of 1000, leading to overcharging users.

**Impact:** 

- **Overcharging users** on swaps.
  
**Recommended Mitigation:** 

Replace magic numbers with constants and use the correct scale.

```diff
function getInputAmountBasedOnOutput(uint256 outputAmount, uint256 inputReserves, uint256 outputReserves
    ) public pure revertIfZero(outputAmount) revertIfZero(outputReserves) returns (uint256 inputAmount)
    {
-     return ((inputReserves * outputAmount) * 10000) / ((outputReserves - outputAmount) * 997);
+     return ((inputReserves * outputAmount) * 1000) / ((outputReserves - outputAmount) * 997);
    }
```

## Low Severity Findings

### [L-1] `TSwap::_addLiquidityMintAndTransfer` function emits `TSwap::LiquidityAdded` with parameters out of order 

**Description:** 
When `LiquidityAdded` is emitted in the `_addLiquidityMintAndTransfer` logs value in an incorrect order. The `poolTokensToDeposit` which is the second parameter should be on the third parameter postion and `wethToDeposit` on the second position. 

**Impact:** 
Event emmision will be incorrect, leading to off-chain functions to potentially malfunction.

**Recommended Mitigation:** 

```diff
- emit LiquidityAdded(msg.sender, poolTokensToDeposit, wethToDeposit);
+ emit LiquidityAdded(msg.sender, wethToDeposit, poolTokensToDeposit);
```

### [L-2] Default value returned by  `TSwapFactory::swapExactInput` function is not updated resulting in an incorrect return value

**Description:** 
The `swapExactInput` function is expected to return the actuall amount of tokens bought by the caller. However, while it declared the named return value `output`, it is never assigned a value nor uses an explicit return statement.

**Impact:** 

- The return value will always be 0, giving incorrect information to the user.

**Recommended Mitigation:** 

```diff
{
        uint256 inputReserves = inputToken.balanceOf(address(this));
        uint256 outputReserves = outputToken.balanceOf(address(this));

-        uint256 outputAmount = getOutputAmountBasedOnInput(inputAmount, inputReserves, outputReserves);
+        output = getOutputAmountBasedOnInput(inputAmount, inputReserves, outputReserves);
        if (outputAmount < minOutputAmount) {
            revert TSwapPool__OutputTooLow(outputAmount, minOutputAmount);
        }

        _swap(inputToken, inputAmount, outputToken, outputAmount);
    }
```

## Informationals

### [I-1] `PoolFactory::PoolFactory__PoolDoesNotExist`is not used and should be removed

```diff
- error PoolFactory__PoolDoesNotExist(address tokenAddress);
```

### [I-2] Lacking 0 address checks

`PoolFactory::contructor`

```diff 
constructor(address wethToken) {
+   if (wethToken == address(0)) revert();
    i_wethToken = wethToken;
    }

```

`TSwapPool::constructor`

```diff
    constructor(address poolToken, address wethToken, string memory liquidityTokenName, string memory liquidityTokenSymbol) 
    ERC20(liquidityTokenName, liquidityTokenSymbol) {
+       if(poolToken == address(0)) revert();
+       if(wethToken == address(0)) revert();
        i_wethToken = IERC20(wethToken);
        i_poolToken = IERC20(poolToken);
    } 
```

### [I-3] `PoolFactory::createPool` should use `symbol()` instead of `name()`

```diff 
- string memory liquidityTokenSymbol = string.concat("ts", IERC20(tokenAddress).name());
+ string memory liquidityTokenSymbol = string.concat("ts", IERC20(tokenAddress).symbol());

```

### [I-4] `PoolFactory::PoolCreated` events is missing an indexed field 

```diff 
- event PoolCreated(address tokenAddress, address poolAddress);
+ event PoolCreated(address indexed tokenAddress, address poolAddress);

```

### [I-5] `TSwapFactory::swapExactInput` function is marked as public and not used internally, it should be updated to external 

```diff 
- function swapExactInput( IERC20 inputToken, uint256 inputAmount, IERC20 outputToken, 
-uint256 minOutputAmount, uint64 deadline ) public 
+ function swapExactInput( IERC20 inputToken, uint256 inputAmount, IERC20 outputToken, 
+ uint256 minOutputAmount, uint64 deadline ) external 

```

### [I-6] `TSwapFactory::revertIfZero` modifier has a strict zero check

```diff 
 modifier revertIfZero(uint256 amount) {
-       if (amount == 0) {
+       if (amount <=  0) {
            revert TSwapPool__MustBeMoreThanZero();
        }
        _;
    }

```
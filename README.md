# Smart Contract Security Audits Repository

Welcome to the centralized repository housing comprehensive security audit reports for multiple blockchain projects. This repository serves as a growing library of audit findings and recommendations, ensuring transparency and trustworthiness across all reviewed smart contracts.

---

## Projects Audited

### 1. PasswordStore Contract Audit (First Report)

The inaugural audit report covers the [PasswordStore](https://github.com/Cyfrin/3-passwordstore-audit/tree/audit-data) smart contract project. This contract allows users to store and update a private password on-chain. The review highlights critical security findings and provides detailed mitigation recommendations.

- **Code Repository:** [PasswordStore GitHub Code](https://github.com/Cyfrin/3-passwordstore-audit/tree/audit-data)  
- **Deployed Contract (Testnet):** [Sepolia Testnet Etherscan](https://sepolia.etherscan.io/address/0x2ecf6ad327776bf966893c96efb24c9747f6694b)  
- **Audit Report:** See `PasswordStore_Security_Audit_Report.md`

---

### 2. PuppyRaffle Contract Audit

The second audit reviews the [PuppyRaffle](https://github.com/Cyfrin/4-puppy-raffle-audit) smart contract, which enables users to participate in a decentralized raffle for a chance to win puppy-themed NFTs. Users pay an entrance fee and can request refunds or await randomized winner selection after a defined duration. NFTs have varying rarity (common, rare, legendary) determined by on-chain randomness. The audit identifies risks including reentrancy, weak randomness, and potential denial-of-service due to inefficient player deduplication.

- **Code Repository:** [PuppyRaffle GitHub Code](https://github.com/Cyfrin/4-puppy-raffle-audit)  
- **Deployed Contract:** Not available at this time  
- **Audit Report:** See `PuppyRaffle-Security-Audit-Report.md`

---

### 3. TSwap Protocol Audit

The third audit covers the [TSwap](https://github.com/Cyfrin/5-t-swap-audit) protocol — a permissionless decentralized exchange (DEX) allowing users to swap assets directly at fair market prices. TSwap follows the Automated Market Maker (AMM) model, similar to Uniswap, using liquidity pools instead of traditional order books.  

**Key Components:**

- **PoolFactory Contract:** Responsible for creating new token pools and ensuring correct logic and configuration.  
- **TSwapPool Contracts:** Each pool facilitates swaps between exactly two assets — an ERC20 token and WETH. Multiple pools allow users to chain swaps across tokens (“token hopping”).

The audit assessed the protocol’s core contracts, swap logic, liquidity mechanisms, and adherence to security best practices.

- **Code Repository:** [TSwap GitHub Code](https://github.com/Cyfrin/5-t-swap-audit)  
- **Deployed Contract:** Not available at this time  
- **Audit Report:** See `TSwap_Smart_Contract_Security_Audit_Report.md`

---

### 4. ThunderLoan Protocol Audit

The latest audit reviews the [ThunderLoan](https://github.com/Cyfrin/6-thunder-loan-audit) smart contract — a flashloan and lending protocol allowing users to borrow and repay ERC20 tokens in a single transaction. The audit identifies potential risks including storage collisions, exchange rate miscalculations, oracle price manipulation, and initialization front-running.

**Key Components:**

- **ThunderLoan Contract:** Manages flashloans, deposits, and fee calculations.  
- **ThunderLoanUpgraded Contract:** Upgradeable implementation with careful attention to storage layout.  
- **Oracle Dependency:** TSwap AMM-based oracle is used for fee and price calculations.  

- **Code Repository:** [ThunderLoan GitHub Code](https://github.com/Cyfrin/6-thunder-loan-audit)  
- **Deployed Contract:** Not available at this time  
- **Audit Report:** See `ThunderLoan-Security-Audit.md`

---

## Future Audits

This repository will be regularly updated with additional audit reports as more projects are reviewed. Upcoming security analyses will follow the same rigorous standards.

---

## Contributing

If you wish to contribute or request an audit for a specific project, please open an issue or contact the repository maintainer.

---

**Report Author:** Philani A Dlamini  
**Date:** September 2025
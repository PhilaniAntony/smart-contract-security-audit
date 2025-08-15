# Smart Contract Security Audits Repository

Welcome to the centralized repository housing comprehensive security audit reports for multiple blockchain projects. This repository will serve as a growing library of audit findings and recommendations, ensuring transparency and trustworthiness across all reviewed smart contracts.

---

## Projects Audited

### 1. PasswordStore Contract Audit (First Report)

The inaugural audit report covers the [PasswordStore](https://github.com/Cyfrin/3-passwordstore-audit/tree/audit-data) smart contract project. This contract allows users to store and update a private password on-chain, and our review highlights critical security findings along with mitigation recommendations.

- **Code Repository:** [PasswordStore GitHub Code](https://github.com/Cyfrin/3-passwordstore-audit/tree/audit-data)  
- **Deployed Contract (Testnet):** [Sepolia Testnet Etherscan](https://sepolia.etherscan.io/address/0x2ecf6ad327776bf966893c96efb24c9747f6694b)  
- **Audit Report:** Detailed findings and recommended mitigations for PasswordStore are available within this repository (see `PasswordStore_Security_Audit_Report.md`).

---

### 2. PuppyRaffle Contract Audit

The second audit reviews the [PuppyRaffle](https://github.com/Cyfrin/4-puppy-raffle-audit) smart contract. This project enables users to participate in a decentralized raffle for a chance to win a puppy-themed NFT. Users pay an entrance fee and can request refunds or await randomized winner selection after a defined duration. NFTs have varying rarity (common, rare, legendary) determined by on-chain randomness. The audit identifies risks including reentrancy, weak randomness, and denial-of-service potential due to inefficient player deduplication.

- **Code Repository:** [PuppyRaffle GitHub Code](https://github.com/Cyfrin/4-puppy-raffle-audit)  
- **Deployed Contract:** Not available at this time  
- **Audit Report:** See `PuppyRaffle-Security-Audit-Report.md`

---

### 3. TSwap Protocol Audit

The third audit covers the [TSwap](https://github.com/Cyfrin/5-t-swap-audit) protocol — a permissionless decentralized exchange (DEX) allowing users to swap assets directly at fair market prices. TSwap follows the Automated Market Maker (AMM) model, similar to Uniswap, using liquidity pools instead of traditional order books.  

**Key Components:**

- **PoolFactory Contract:** Responsible for creating new token pools, ensuring each pool follows correct logic and configuration.
- **TSwapPool Contracts:** Each pool facilitates swaps between exactly two assets — an ERC20 token and WETH. Once enough pools exist, users can chain swaps across multiple tokens (“token hopping”).

The audit assessed the protocol’s core contracts, swap logic, liquidity mechanisms, and adherence to security best practices.

- **Code Repository:** [TSwap GitHub Code](https://github.com/Cyfrin/5-t-swap-audit)  
- **Deployed Contract:** Not available at this time  
- **Audit Report:** See `TSwap_Smart_Contract_Security_Audit_Report.md`

---

## Future Audits

This repository will be regularly updated with additional audit reports as more projects are reviewed. Stay tuned for upcoming security analyses covering the next five projects, each assessed with the same rigorous standards.

---

## Contributing

If you wish to contribute or request an audit for a specific project, please open an issue or contact the repository maintainer.

---

**Report Author:** Philani A Dlamini  
**Date:** August 2025

---
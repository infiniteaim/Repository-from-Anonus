# SupplyChainTracker - Design Choices & Assumptions

## **Core Purpose**
A role-based supply chain tracking system where:
- **Manufacturer**, **Distributor**, and **Retailer** update product statuses.
- Participants earn rewards (1 token per update) for valid actions.

---

## **Key Design Choices**

### 1. **Access Control**
- **Ownable**: Admin (deployer) can transfer ownership if needed.
- **Role-Based Modifiers**:
  - `onlyRole()` ensures only assigned addresses (manufacturer/distributor/retailer) can call `updateStatus`.
  - Roles are **hardcoded at deployment** (no dynamic role assignment).

### 2. **Security**
- **ReentrancyGuard**: Prevents reentrancy attacks on `updateStatus` (though no external calls are made here, it’s defensive).
- **Input Validation**: Empty status updates are rejected.
- **No External Calls in Critical Paths**: Avoids reentrancy risks (e.g., no token transfers in `updateStatus`).

### 3. **Data Storage**
- **Status History**: Stored in a private `string[]` (emitted via events for off-chain indexing).
- **Rewards**: Tracked in a `mapping` (not an ERC20 token; assumes rewards are claimed later via a separate mechanism).

### 4. **Gas Efficiency**
- **View Functions**: `getHistory()` and `getRewardBalance()` are `view`/`pure` to avoid gas costs for read-only ops.
- **Minimal Storage Writes**: Only appends to arrays/mappings when necessary.

### 5. **Assumptions & Limitations**
- **Static Roles**: Roles cannot be reassigned post-deployment (would require an `onlyOwner` function to update).
- **Reward System**: Assumes rewards are **virtual** (no ERC20 integration). A real implementation might:
  - Use OpenZeppelin’s `ERC20` for actual token rewards.
  - Add a `claimRewards()` function to transfer tokens.
- **No Batch Operations**: Updates are single-status (no bulk updates).
- **No Upgradability**: Contract is not upgradeable (consider `UUPS` or `Transparent Proxy` if needed).

### 6. **Events**
- **`StatusUpdated`**: Logs operator, new status, reward amount, and timestamp for off-chain tracking.
- **`RewardClaimed`**: Placeholder for future reward claims (currently unused).

# EnergyCreditMarket - Design Choices & Assumptions

## **Core Purpose**
A marketplace for **energy credits**, where:
- **Producers** (e.g., renewable energy farms) mint credits.
- **Consumers** buy/redeem credits to offset carbon footprints.
- **Owner** manages producer registration and credit pricing.

---

## **Key Design Choices**

### 1. **Roles & Permissions**
- **Owner**: Admin role (via `Ownable`) that:
  - Registers producers (`registerProducer`).
  - Updates credit price (`updatePrice`).
- **Producers**: Whitelisted addresses that can **mint** credits.
- **Consumers**: Any address that can **buy/transfer/redeem** credits.

### 2. **Credit Lifecycle**
1. **Minting**: Producers create credits (increases `totalSupply`).
2. **Buying**: Consumers purchase credits at `pricePerCredit` (payable in ETH).
3. **Transferring**: Consumers can send credits to other addresses.
4. **Redeeming**: Consumers burn credits (decreases `totalSupply`, increases `totalRedeemed`).

### 3. **Security**
- **Reentrancy Mitigation**: No external calls in state-changing functions (e.g., no `transfer` after `buyCredits`).
- **Input Validation**:
  - Zero-address checks (`registerProducer`, `transferCredits`).
  - Zero-amount checks (`mintCredits`, `buyCredits`, etc.).
  - Balance checks before transfers/redemptions.
- **Ownable Critical Functions**: Only owner can register producers or set prices.

### 4. **Data Storage**
- **Producers**: Tracked in a `mapping` with `isRegistered` flag and `balance`.
- **Consumers**: Balances stored in `consumerBalances` mapping.
- **Market Stats**: `totalSupply`, `totalRedeemed`, and `pricePerCredit` are public variables.

### 5. **Payment Handling**
- **ETH-Based**: Consumers pay in ETH for credits (`buyCredits` is `payable`).
- **Change Refund**: Excess ETH is refunded via `transfer` (note: uses division, which may not fully refund due to integer truncation).

### 6. **Assumptions & Limitations**
- **No ERC20 Token**: Credits are **virtual** (not fungible tokens). A real implementation might:
  - Use `ERC20` for interoperability (e.g., trading on DEXs).
  - Add `safeTransfer` checks for contract recipients.
- **Fixed-Point Math**: Uses basic arithmetic (no decimals; assumes `amount` is in whole units).
- **No Batch Operations**: Single-credit operations only (e.g., no bulk mint/transfer).
- **No Pausable/Emergency Stop**: Contract cannot be paused in case of exploits.
- **Refund Logic Flaw**: `msg.value / totalCost` in `buyCredits` **incorrectly calculates change** (should subtract `totalCost` from `msg.value`).

### 7. **Events**
- **Transparency**: All critical actions emit events for off-chain tracking (e.g., `CreditsMinted`, `CreditsRedeemed`).
- **Market Activity**: `PriceUpdated` logs price changes.

### 8. **Gas Efficiency**
- **View Functions**: `getMarketSummary` is `view` for gas-free reads.
- **Minimal Storage Writes**: Only updates necessary mappings/variables.

   if (msg.value > totalCost) {
       payable(msg.sender).transfer(msg.value - totalCost); // Subtract, not divide
   }

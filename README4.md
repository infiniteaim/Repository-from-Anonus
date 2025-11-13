# MultiSigEscrow - Design Choices & Assumptions

## **Core Purpose**
A **2-of-3 multisig escrow** contract for secure ETH transactions between a **buyer** and **seller**, with a **mediator** to resolve disputes. Funds are released only if **at least 2 participants approve**.

---

## **Key Design Choices**

### 1. **Participants & Roles**
- **Buyer**: Deposits ETH into escrow.
- **Seller**: Receives funds after approval.
- **Mediator**: Neutral third party to resolve disputes.
- **Unique Addresses**: All 3 roles must be distinct (enforced in constructor).

### 2. **Escrow Workflow**
1. **Deployment**: Buyer, seller, and mediator addresses are set.
2. **Deposit**: Buyer sends ETH to the contract (`deposit`).
3. **Approval**: Any 2 of the 3 participants call `approveRelease`.
4. **Release**: After 2 approvals, funds are sent to the seller (`finalizeRelease`).

### 3. **State Management**
- **`EscrowState` Enum**:
  - `Pending`: Initial state (awaiting deposit/approvals).
  - `Approved`: 2+ approvals received.
  - `Released`: Funds sent to seller.
- **State Modifiers**: Functions like `deposit`/`approveRelease` can only be called in specific states (e.g., `inState(EscrowState.Pending)`).

### 4. **Security**
- **Access Control**:
  - `onlyParticipants`: Restricts critical functions to buyer/seller/mediator.
  - `notAlreadyApproved`: Prevents duplicate approvals.
- **Reentrancy Protection**: Uses `call` for ETH transfer (not `transfer`/`sendValue`) with no prior external calls.
- **Input Validation**:
  - Zero-address checks in constructor.
  - Non-zero deposit amount.
  - State checks before operations.

### 5. **Approval Mechanism**
- **2-of-3 Multisig**: Requires 2 approvals to release funds.
- **Tracking**: `hasApproved` mapping and `approvalCount` ensure no double-counting.

### 6. **Assumptions & Limitations**
- **ETH-Only**: Only handles ETH (not ERC20 tokens).
- **No Timeouts**: Funds can be locked indefinitely if participants don’t approve.
- **No Refund Path**: If approvals fail, funds remain stuck (no `cancel` function).
- **Mediator Trust**: Mediator has equal power to buyer/seller (could collude).
- **No Batch Approvals**: Participants must approve individually.
- **Front-Running Risk**: Approvals are visible on-chain (could influence mediator).

### 7. **Key Functions**
| Function            | Access               | Description                                  |
|---------------------|----------------------|----------------------------------------------|
| `deposit`           | Buyer (payable)      | Locks ETH in escrow.                        |
| `approveRelease`    | Participants         | Approves fund release (max 1x per participant). |
| `finalizeRelease`   | Anyone               | Releases ETH to seller after 2 approvals.   |
| `getEscrowStatus`   | Public (view)        | Returns human-readable state.               |
| `getDepositAmount`  | Public (view)        | Returns deposited ETH amount.               |

### 8. **Events**
- **Transparency**: Critical actions emit events:
  - `DepositReceived`: Buyer’s ETH deposited.
  - `ApprovalReceived`: Participant approves release.
  - `FundsReleased`: ETH sent to seller.

### 9. **Gas Efficiency**
- **Minimal Storage**: Only essential variables (`approvalCount`, `hasApproved`, etc.).
- **View Functions**: `getEscrowStatus` and `getDepositAmount` are gas-free.

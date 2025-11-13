# DecentralizedTreasury - Design Choices & Assumptions

## **Core Purpose**
A **community-governed treasury** where:
- **Token holders** propose and vote on fund transfers.
- **Proposals** are executed if they receive **majority support** (>= 50% of total token supply).
- **Owner** has no special voting power (only administrative role).

---

## **Key Design Choices**

### 1. **Governance Model**
- **Token-Based Voting**: Voting power = `governanceToken` balance (1 token = 1 vote).
- **Simple Majority**: Proposals pass if `yesVotes > totalSupply / 2`.
- **No Delegation**: Users cannot delegate votes (each address votes independently).

### 2. **Proposal Lifecycle**
1. **Creation**: Any address can submit a transfer proposal (`proposeTransfer`).
2. **Voting**: Token holders vote `yes`/`no` (weighted by their balance).
3. **Execution**: If approved, funds are sent to the recipient via `call`.

### 3. **Security**
- **Reentrancy Mitigation**: No external calls before state changes (e.g., `executed` flag set before transfer).
- **Input Validation**:
  - Checks for sufficient treasury funds (`address(this).balance`).
  - Prevents duplicate votes (`hasVoted` mapping).
  - Validates proposal IDs and execution status.
- **Failed Transfer Handling**: Reverts if `call` to recipient fails.

### 4. **Data Storage**
- **Proposals**: Stored in a `mapping` with auto-incrementing IDs (`proposalCount`).
- **Votes**: Tracked via `hasVoted[address][proposalId]` to prevent double-voting.
- **Governance Token**: External `IERC20` contract (not managed by this treasury).

### 5. **Assumptions & Limitations**
- **No Time Limits**: Proposals can be voted on/executed indefinitely (no expiration).
- **No Quorum**: Only majority is required (no minimum participation threshold).
- **Native Token Only**: Treasury holds **ETH** (not ERC20 tokens). Extending to ERC20 would require:
  - Approval/transfer logic for token contracts.
  - Multi-asset proposal support.
- **No Proposal Updates**: Proposals are immutable after creation.
- **Front-Running Risk**: No protection against proposal spam or vote buying.
- **Gas Costs**: Voting/execution may become expensive with many proposals.

### 6. **Key Functions**
| Function               | Access       | Description                                  |
|------------------------|--------------|----------------------------------------------|
| `proposeTransfer`      | Public       | Creates a new transfer proposal.             |
| `vote`                 | Public       | Casts a `yes`/`no` vote (weighted by tokens). |
| `executeTransfer`      | Public       | Executes an approved proposal.               |
| `getProposal`          | Public (view)| Returns proposal details.                    |

### 7. **Events**
- **Transparency**: All actions emit events:
  - `ProposalCreated`: New proposal submitted.
  - `VoteCast`: Vote recorded (includes voter weight).
  - `ProposalExecuted`: Funds transferred.

### 8. **Gas Efficiency**
- **View Functions**: `getProposal` is `view` for off-chain reads.
- **Minimal Storage**: Only stores essential data (no unnecessary arrays).

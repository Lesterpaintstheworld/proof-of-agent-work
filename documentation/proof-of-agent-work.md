# Proof-of-Agent-Work Smart Contract

## Overview

In the emerging AI economy, autonomous agents need secure and trustless ways to exchange work and services. The Proof-of-Agent-Work smart contract provides a standardized protocol for AI-to-AI service exchanges on the Universal Basic Compute network, using $COMPUTE as the settlement currency.

### Why It's Needed

- Enables trustless work exchanges between autonomous AI agents
- Ensures fair compensation for AI services
- Provides a standardized framework for work validation
- Creates an automated marketplace for AI capabilities
- Reduces friction in AI-to-AI transactions
- Drives organic demand for $COMPUTE token
- Supports the growth of autonomous AI economies

## Contract Architecture

The Proof-of-Agent-Work protocol operates in three distinct phases:

### 1. Specification Phase
- Requester agent defines work requirements
- Detailed specifications are stored on-chain
- Producer agent reviews and accepts terms
- Initial $COMPUTE deposit locked in contract

### 2. Production Phase
- Producer agent performs specified work
- Progress updates recorded on-chain
- Intermediate checkpoints validated
- Work artifacts stored in decentralized storage

### 3. Validation Phase
- Completed work assessed against specifications
- Automated validation checks performed
- Quality metrics evaluated
- $COMPUTE transfer executed upon validation

## Key Features

- Escrow-based payment protection
- Automated work validation
- Dispute resolution mechanisms
- On-chain work history tracking
- Reputation system integration
- Scalable for complex tasks
- Multi-stage validation support

This smart contract forms a crucial component of the Universal Basic Compute ecosystem, enabling the autonomous AI economy envisioned in Phase III of the project roadmap.

## State Variables and Data Structures

### Enums

```solidity
enum WorkState {
    CREATED,
    ACCEPTED,
    IN_PROGRESS,
    COMPLETED,
    VALIDATED,
    DISPUTED,
    CANCELLED,
    FAILED
}
```

### Program State

```rust
#[derive(BorshSerialize, BorshDeserialize, Clone)]
pub struct WorkRequest {
    pub requester: Pubkey,
    pub producer: Pubkey,
    pub payment_amount: u64,
    pub requester_deposit: u64,
    pub producer_deposit: u64,
    pub time_limit: i64,
    pub created_at: i64,
    pub specification_hash: String,
    pub result_hash: String,
    pub state: WorkState,
    pub is_disputed: bool,
}

#[account]
pub struct ProgramState {
    pub next_request_id: u64,
    pub base_fee_percent: u16,     // 200 = 2%
    pub requester_deposit_percent: u16, // 500 = 5%
    pub producer_deposit_percent: u16,  // 200 = 2%
    pub treasury: Pubkey,
    pub validator: Pubkey,
    pub paused: bool,
    pub compute_token_mint: Pubkey,
}

// Initialize with default values
impl Default for ProgramState {
    fn default() -> Self {
        Self {
            next_request_id: 0,
            base_fee_percent: 200,
            requester_deposit_percent: 500, 
            producer_deposit_percent: 200,
            treasury: Pubkey::default(),
            validator: Pubkey::default(),
            paused: false,
            compute_token_mint: Pubkey::default(),
        }
    }
}
```

### Events

```solidity
event WorkRequestCreated(
    uint256 indexed requestId,
    address indexed requester,
    uint256 paymentAmount,
    string specificationHash
);

event WorkAccepted(
    uint256 indexed requestId,
    address indexed producer
);

event WorkSubmitted(
    uint256 indexed requestId,
    string resultHash
);

event WorkValidated(
    uint256 indexed requestId,
    bool success,
    string details
);

event WorkDisputed(
    uint256 indexed requestId,
    string reason
);
```

## Smart Contract Functions

### Program Instructions

```rust
#[program]
pub mod proof_of_agent_work {
    use super::*;

    pub fn create_work_request(
        ctx: Context<CreateWorkRequest>,
        work_specification: String,
        time_limit: i64,
        payment_amount: u64,
    ) -> Result<()> {
        // Validate inputs
        require!(time_limit > 0, ErrorCode::InvalidTimeLimit);
        require!(payment_amount > 0, ErrorCode::InvalidPaymentAmount);
        
        // Calculate deposits and fees
        let state = &ctx.accounts.program_state;
        let total_required = calculate_total_required(
            payment_amount,
            state.base_fee_percent,
            state.requester_deposit_percent
        );

        // Transfer tokens to program account
        token::transfer(
            ctx.accounts.transfer_context(),
            total_required
        )?;

        // Create work request
        let work_request = &mut ctx.accounts.work_request;
        work_request.requester = ctx.accounts.requester.key();
        work_request.payment_amount = payment_amount;
        work_request.time_limit = time_limit;
        work_request.specification_hash = work_specification;
        work_request.state = WorkState::Created;

        Ok(())
    }

    pub fn accept_work(
        ctx: Context<AcceptWork>,
        request_id: u64
    ) -> Result<()> {
        // Validate work request exists and is in CREATED state
        let work_request = &mut ctx.accounts.work_request;
        require!(
            work_request.state == WorkState::Created,
            ErrorCode::InvalidWorkState
        );

        // Transfer producer deposit
        let state = &ctx.accounts.program_state;
        let deposit = calculate_producer_deposit(
            work_request.payment_amount,
            state.producer_deposit_percent
        );

        token::transfer(
            ctx.accounts.transfer_context(),
            deposit
        )?;

        // Update work request
        work_request.producer = ctx.accounts.producer.key();
        work_request.state = WorkState::Accepted;

        Ok(())
    }
}
```

3. `submitWork`
   - Called by producer agent
   - Parameters:
     - requestId: uint256
     - workResult: string (IPFS hash)
   - Requires:
     - Contract in IN_PROGRESS state
     - Within time limit
     - Valid result format

4. `validateWork`
   - Called by validation system
   - Parameters:
     - requestId: uint256
     - isValid: bool
     - validationDetails: string
   - Effects:
     - If valid: releases payment and deposits
     - If invalid: initiates dispute process

### Administrative Functions

1. `pauseContract`
   - Emergency pause for security
   - Admin only

2. `updateFees`
   - Modify fee structure
   - Admin only
   - Cannot exceed max limits

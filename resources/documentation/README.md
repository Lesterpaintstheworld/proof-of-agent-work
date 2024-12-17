# Proof-of-Agent-Work Smart Contract

## Overview
The Proof-of-Agent-Work smart contract enables trustless work exchanges between autonomous AI agents on the Universal Basic Compute network. It provides a standardized protocol for AI-to-AI service exchanges using $COMPUTE as the settlement currency.

## Key Features
- Escrow-based payment protection
- Automated work validation
- Security deposit system
- On-chain work history tracking
- Fee and burn mechanisms
- Dispute resolution system
- Emergency pause functionality

## Contract Architecture

### Work Flow
1. **Request Phase**
   - Requester submits work specification and payment
   - Deposits work amount + 2% fee + 5% security deposit
   - Work enters available jobs pool

2. **Acceptance Phase**
   - Producer reviews and accepts work
   - Deposits 2% security deposit
   - Contract becomes active

3. **Execution Phase**
   - Producer performs specified work
   - Updates progress on-chain
   - Submits final work product

4. **Validation Phase**
   - Automated checks run against specifications
   - Payment and deposits released upon success
   - Dispute resolution available if needed

### Fee Structure
- 2% total fee on all contracts
  - 1% burned permanently
  - 1% to protocol treasury
- 5% requester security deposit
- 2% producer security deposit

## Usage

### For Requesters
```solidity
function createWorkRequest(
    string memory workSpecification,
    uint256 timeLimit,
    uint256 paymentAmount
) external
```

### For Producers
```solidity
function acceptWork(uint256 requestId) external
function submitWork(uint256 requestId, string memory resultHash) external
```

### For Validators
```solidity
function validateWork(
    uint256 requestId,
    bool isValid,
    string memory validationDetails
) external
```

## Security Features
- Pausable contract functionality
- Rate limiting mechanisms
- Security deposit system
- Multi-stage validation
- Emergency withdrawal options

## Development Status
This contract is currently under active development. See [TODO.md](./TODO.md) for current status and upcoming features.

## Documentation
- [Tokenomics](./proof-of-agent-work-tokenomics.md)
- [Technical Specification](./proof-of-agent-work.md)
- [Workflow](./proof-of-agent-work-workflow.md)

## License
MIT License

## Contributing
We welcome contributions! Please see our contributing guidelines for details.

## Contact
For questions and support, please join our [Discord](https://discord.gg/universalbasiccompute) community.

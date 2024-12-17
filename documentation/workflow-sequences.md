# Proof-of-Agent-Work Workflow Sequences

## Core Transaction Flow

### 1. Work Request Creation
```mermaid
sequenceDiagram
    participant R as Requester Agent
    participant C as Contract
    participant T as Treasury
    
    R->>C: createWorkRequest()
    Note over R,C: Includes payment + deposits
    C->>T: Transfer base fee
    C->>C: Lock deposits & payment
    C->>R: Return requestId
```

### 2. Work Acceptance & Production
```mermaid
sequenceDiagram
    participant P as Producer Agent
    participant C as Contract
    participant S as Storage
    
    P->>C: acceptWork()
    Note over P,C: Includes producer deposit
    P->>S: Store work artifacts
    P->>C: submitWork()
    Note over P,C: Includes result hash
```

### 3. Validation & Settlement
```mermaid
sequenceDiagram
    participant V as Validator
    participant C as Contract
    participant P as Producer
    participant R as Requester
    
    V->>C: validateWork()
    alt Success
        C->>P: Transfer payment
        C->>P: Return deposit
        C->>R: Return deposit
    else Failure
        C->>R: Return payment
        Note over C: Handle deposits per dispute
    end
```

## State Transitions

### Valid State Changes
```
CREATED -> ACCEPTED -> IN_PROGRESS -> COMPLETED -> VALIDATED
                                               \-> DISPUTED
DISPUTED -> VALIDATED | FAILED
ANY_STATE -> CANCELLED (admin only)
```

### State Change Requirements

1. CREATED -> ACCEPTED
   - Valid producer address
   - Producer deposit provided
   - Within acceptance timeframe
   - Contract not paused

2. ACCEPTED -> IN_PROGRESS
   - Producer confirmed
   - Time limit not exceeded
   - Required deposits locked

3. IN_PROGRESS -> COMPLETED
   - Work result hash provided
   - Within time limit
   - Valid result format

4. COMPLETED -> VALIDATED
   - Validation checks passed
   - No active disputes
   - Validator consensus met

### Error Handling

1. Insufficient Funds
   ```
   if (token.balanceOf(msg.sender) < totalRequired)
       revert InsufficientFunds(required, available)
   ```

2. Invalid State Transition
   ```
   if (!isValidTransition(currentState, newState))
       revert InvalidStateTransition(currentState, newState)
   ```

3. Time Limit Exceeded
   ```
   if (block.timestamp > request.timeLimit)
       revert TimeLimitExceeded(timeLimit, block.timestamp)
   ```

## Validation Requirements

### Basic Validation Checks
1. Result hash format verification
2. Time limit compliance
3. Deposit requirements met
4. State transition validity

### Advanced Validation
1. Work quality assessment
2. Resource usage verification
3. Output format compliance
4. Performance metrics

### Dispute Triggers
1. Invalid result format
2. Incomplete deliverables
3. Quality below threshold
4. Time limit violation
5. Resource misuse

## Security Considerations

### Access Control
- Function-level permissions
- State-dependent restrictions
- Admin capabilities
- Emergency controls

### Economic Security
- Minimum deposit requirements
- Fee calculations
- Slashing conditions
- Treasury operations

### Technical Security
- Reentrancy protection
- Integer overflow checks
- Gas optimization
- State consistency

## Integration Points

### External Interfaces
1. Token contracts (COMPUTE)
2. Storage systems (IPFS)
3. Validation nodes
4. Treasury management

### Event Emissions
1. State changes
2. Financial operations
3. Validation results
4. Dispute handling

### Monitoring Hooks
1. Transaction tracking
2. State monitoring
3. Economic metrics
4. System health

---
title: EVM Bytecode Structure
date: 2025-12-23 10:30 +0700
categories: [Blockchain, EVM]
tags: [evm, bytecode, solidity, assembly, smart-contracts]
author: mxzyy
---
## Bytecode Structure Overview

When you compile Solidity code, the compiler produces bytecode - the low-level instructions the EVM executes. This bytecode isn't just a random sequence of opcodes. The Solidity compiler organizes it into a specific structure that handles function routing, execution, and metadata storage.

Understanding bytecode structure helps with gas optimization, security auditing, and debugging contracts at the opcode level.


Compiled Solidity contracts follow this general layout:

```
┌──────────────────────────────────────────────┐
│  1. CREATION CODE (deployed once)            │
├──────────────────────────────────────────────┤
│  2. RUNTIME CODE (stored on-chain)           │
│     - Preamble / Initialization              │
│     - Function Dispatcher                    │
│     - Function Bodies (Public/External)      │
│     - Internal Functions                     │
│     - Utility & Error Handlers               │
│     - Metadata Hash                          │
└──────────────────────────────────────────────┘
```

**Creation code** runs once during deployment and returns the **runtime code**, which is stored on-chain.

The runtime code is what executes when you call the deployed contract.

---

## 1. Preamble (Free Memory Pointer Setup)

The first few opcodes in runtime bytecode initialize the **Free Memory Pointer (FMP)** and perform basic validation.

**Common preamble pattern:**

```
PUSH1 0x80        // Push 0x80 to stack
PUSH1 0x40        // Push 0x40 to stack
MSTORE            // Store 0x80 at memory location 0x40
```

This sets the Free Memory Pointer to `0x80`, establishing where free memory starts (see [Layout Memory](https://mxzyy.github.io/posts/Layout-Memory/) for details).

**Non-payable check:**

```
CALLVALUE         // Get msg.value
DUP1              // Duplicate it
ISZERO            // Check if zero
PUSH2 0x0010      // Jump destination if zero
JUMPI             // Conditional jump
PUSH1 0x00        // If msg.value != 0
DUP1
REVERT            // Revert transaction
JUMPDEST          // Valid jump target
POP               // Clean up stack
```

This rejects ETH sent to non-payable functions.

---

## 2. Function Dispatcher

The dispatcher is a routing mechanism that reads the **function selector** (first 4 bytes of calldata) and jumps to the corresponding function code.

**How it works:**

1. Load calldata size and check if >= 4 bytes
2. Extract first 4 bytes (function selector)
3. Compare selector against known function signatures
4. Jump to matching function code
5. Revert if no match found

**Example dispatcher logic:**

```
PUSH1 0x04           // Push 4
CALLDATASIZE         // Get calldata size
LT                   // Check if calldata < 4 bytes
PUSH2 0x003f         // Jump destination if invalid
JUMPI                // Jump if calldata too short

PUSH1 0x00           // Offset 0
CALLDATALOAD         // Load 32 bytes from calldata
PUSH1 0xe0           // Push 224 (bits to shift)
SHR                  // Right shift to get first 4 bytes

DUP1                 // Duplicate selector
PUSH4 0xa9059cbb     // Selector for transfer(address,uint256)
EQ                   // Compare
PUSH2 0x0045         // Jump destination for transfer
JUMPI                // Jump if match

DUP1                 // Duplicate selector again
PUSH4 0x70a08231     // Selector for balanceOf(address)
EQ                   // Compare
PUSH2 0x0067         // Jump destination for balanceOf
JUMPI                // Jump if match

REVERT               // No match, revert
```

The dispatcher uses a **linear search** or **binary search** pattern depending on the number of functions.

---

## 3. Function Bodies (Public/External)

Each public or external function has its own bytecode section that:

1. Decodes parameters from calldata
2. Executes function logic
3. Encodes return values
4. Returns or reverts

**Example: Simple getter function**

```solidity
uint256 public value;
```

Compiles to (simplified):

```
JUMPDEST              // Valid jump target
PUSH1 0x00            // Storage slot 0
SLOAD                 // Load value from storage
PUSH1 0x40            // Free memory pointer location
MLOAD                 // Load FMP
DUP1                  // Duplicate FMP
SWAP2                 // Rearrange stack
DUP3                  // Prepare for MSTORE
MSTORE                // Store value in memory
PUSH1 0x20            // Return data size (32 bytes)
ADD                   // Calculate end of return data
PUSH1 0x40            // Start of return data
MLOAD                 // Load starting position
DUP1                  // Duplicate
SWAP2                 // Rearrange
SUB                   // Calculate size
SWAP1                 // Prepare for RETURN
RETURN                // Return 32 bytes
```

**Example: Function with parameters**

```solidity
function setValue(uint256 _value) public {
    value = _value;
}
```

Compiles to (simplified):

```
JUMPDEST              // Valid jump target
PUSH1 0x04            // Calldata offset (skip selector)
CALLDATALOAD          // Load first parameter (32 bytes)
PUSH1 0x00            // Storage slot 0
SSTORE                // Store to storage
STOP                  // End execution
```

---

## 4. Internal Functions

Internal and private functions are compiled as separate code blocks that public functions can **JUMP** to.

Unlike external calls, internal calls use the EVM stack directly without encoding/decoding calldata.

**Example:**

```solidity
function publicFunc(uint256 x) public returns (uint256) {
    return _internalHelper(x);
}

function _internalHelper(uint256 x) internal pure returns (uint256) {
    return x * 2;
}
```

The bytecode for `_internalHelper` sits in a separate section:

```
// _internalHelper code block
JUMPDEST              // Internal function entry point
PUSH1 0x02            // Push constant 2
DUP2                  // Duplicate parameter x
MUL                   // x * 2
SWAP1                 // Prepare stack
JUMP                  // Return to caller
```

When `publicFunc` calls `_internalHelper`, it uses:

```
PUSH2 0x00a5          // Return address (where to resume)
PUSH2 0x0120          // Jump to _internalHelper
JUMP                  // Execute internal function
JUMPDEST              // Resume here after return
```

This is more gas-efficient than external calls because there's no ABI encoding overhead.

---

## 5. Utility & Error Handlers

This section contains:

- **ABI encoding/decoding helpers** - Shared code for packing/unpacking data
- **Require/revert handlers** - Error message encoding
- **Custom errors** - Error selector + encoded data

**Example: Require with error message**

```solidity
require(balance >= amount, "Insufficient balance");
```

Compiles to:

```
DUP2                  // Duplicate balance
DUP2                  // Duplicate amount
LT                    // balance < amount?
ISZERO                // Invert (we want >=)
PUSH2 0x0045          // Jump if valid
JUMPI                 // Skip revert if condition met

PUSH1 0x40            // Load FMP
MLOAD                 // Get free memory pointer
DUP1                  // Duplicate
PUSH4 0x08c379a0      // Error selector for Error(string)
...                   // Encode error string
REVERT                // Revert with error data
JUMPDEST              // Continue if no error
```

**Custom errors** (more gas-efficient):

```solidity
error InsufficientBalance(uint256 available, uint256 required);

revert InsufficientBalance(balance, amount);
```

Compiles to:

```
PUSH1 0x40            // Load FMP
MLOAD                 // Get memory pointer
PUSH4 0x12ab34cd      // Custom error selector
DUP2                  // Prepare memory location
MSTORE                // Store selector
DUP3                  // Load 'available'
PUSH1 0x04            // Offset for first parameter
DUP3                  // Memory location
ADD                   // Calculate position
MSTORE                // Store first parameter
DUP2                  // Load 'required'
PUSH1 0x24            // Offset for second parameter
DUP3                  // Memory location
ADD                   // Calculate position
MSTORE                // Store second parameter
PUSH1 0x44            // Total data size
PUSH1 0x00            // Start of data
REVERT                // Revert with custom error
```

---

## 6. Metadata

The very end of runtime bytecode contains **CBOR-encoded metadata**:

```
a264697066735822<IPFS_HASH>64736f6c63<COMPILER_VERSION>0033
```

This includes:
- IPFS hash of source code (for verification on Etherscan)
- Solidity compiler version
- Additional compilation settings

**Example:**

```
a2646970667358221220<32-byte-ipfs-hash>64736f6c634300081a0033
```

Breakdown:
- `a2` - CBOR map with 2 entries
- `64697066735822` - "ipfs" key + length
- `1220` - IPFS multihash prefix
- `<32 bytes>` - IPFS hash
- `64736f6c6343` - "solc" key
- `00081a` - Version 0.8.26
- `0033` - End marker

The EVM never executes this section. It's purely informational and used for source verification.

---

## Execution Flow

When a transaction calls a contract:

```
1. Preamble executes
   ├── Set up FMP
   └── Check msg.value (if non-payable)

2. Function Dispatcher
   ├── Extract function selector
   ├── Match against known selectors
   └── JUMP to function code

3. Function Body
   ├── Decode calldata parameters
   ├── Execute logic (may call internal functions via JUMP)
   ├── Encode return data
   └── RETURN or REVERT

4. If error → Jump to error handler → REVERT
   If success → RETURN with data
```

Metadata is never executed - it's dead code at the end.

---

## Practical Example: Counter Contract

```solidity
contract Counter {
    uint256 public count;

    function increment() public {
        count++;
    }
}
```

**Bytecode structure:**

```
┌─────────────────────────────────────────────┐
│ PREAMBLE                                    │
│   PUSH1 0x80, PUSH1 0x40, MSTORE            │
│   (Set FMP to 0x80)                         │
├─────────────────────────────────────────────┤
│ DISPATCHER                                  │
│   Extract selector from calldata            │
│   Compare with:                             │
│     0x06661abd (count())                    │
│     0xd09de08a (increment())                │
│   JUMPI to matched function                 │
├─────────────────────────────────────────────┤
│ count() BODY                                │
│   JUMPDEST                                  │
│   PUSH1 0x00    // slot 0                   │
│   SLOAD         // load count               │
│   MSTORE        // store in memory          │
│   RETURN        // return 32 bytes          │
├─────────────────────────────────────────────┤
│ increment() BODY                            │
│   JUMPDEST                                  │
│   PUSH1 0x00    // slot 0                   │
│   DUP1          // duplicate slot           │
│   SLOAD         // load current count       │
│   PUSH1 0x01    // constant 1               │
│   ADD           // count + 1                │
│   SWAP1         // prepare for SSTORE       │
│   SSTORE        // store new count          │
│   STOP          // end                      │
├─────────────────────────────────────────────┤
│ METADATA                                    │
│   a264697066735822...                       │
└─────────────────────────────────────────────┘
```

---

## Why This Matters

Understanding bytecode structure is essential for:

1. **Gas optimization** - Identify inefficient opcode patterns
2. **Security auditing** - Spot unusual control flow or hidden functions
3. **Debugging** - Trace execution when Solidity-level debugging isn't enough
4. **Reverse engineering** - Analyze contracts without source code
5. **Writing Yul/Assembly** - Know how your code fits into the larger bytecode

The structure is predictable and logical. The preamble initializes, the dispatcher routes calls, function bodies execute logic, and metadata provides verification data. Each section has a clear purpose in the contract's execution model.

---
title: EVM Opcodes
date: 2025-12-23 05:08 +0700
categories: [Blockchain, EVM]
tags: [evm, opcodes, ethereum, assembly, yul, solidity]
author: mxzyy
---

EVM opcodes are the fundamental building blocks of smart contract execution on Ethereum. Think of them as the low-level instructions that the Ethereum Virtual Machine understands and executes. When you write Solidity code, it gets compiled down to bytecode made up of these opcodes. Understanding opcodes is essential for gas optimization, debugging contracts, writing inline assembly, and really grasping how your smart contracts work under the hood. This guide covers all 148 valid opcodes, organized by category, with their hex values, gas costs, and practical descriptions to help you master EVM programming.

## Reference: [evm.codes](https://www.evm.codes) | Ethereum Yellow Paper

---

## 🔢 1. ARITHMETIC OPERATIONS

These opcodes handle basic math operations on 256-bit values.

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| ADD | 0x01 | 3 | Addition: `a + b` (mod 2²⁵⁶) |
| MUL | 0x02 | 5 | Multiplication: `a * b` (mod 2²⁵⁶) |
| SUB | 0x03 | 3 | Subtraction: `a - b` (mod 2²⁵⁶) |
| DIV | 0x04 | 5 | Unsigned division: `a / b` |
| SDIV | 0x05 | 5 | Signed division (int256) |
| MOD | 0x06 | 5 | Unsigned modulus: `a % b` |
| SMOD | 0x07 | 5 | Signed modulus (int256) |
| ADDMOD | 0x08 | 8 | Modular addition: `(a + b) % N` |
| MULMOD | 0x09 | 8 | Modular multiplication: `(a * b) % N` |
| EXP | 0x0A | 10* | Exponentiation: `a ** b` (dynamic gas) |
| SIGNEXTEND | 0x0B | 5 | Sign extension from (b+1) bytes to 32 bytes |

---

## ⚖️ 2. COMPARISON OPERATIONS

These opcodes compare values and return boolean results (0 or 1).

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| LT | 0x10 | 3 | Less than (unsigned): `a < b` |
| GT | 0x11 | 3 | Greater than (unsigned): `a > b` |
| SLT | 0x12 | 3 | Signed less than (int256) |
| SGT | 0x13 | 3 | Signed greater than (int256) |
| EQ | 0x14 | 3 | Equality check: `a == b` |
| ISZERO | 0x15 | 3 | Zero check: `a == 0` |

---

## 🔣 3. BITWISE OPERATIONS

These opcodes manipulate bits at the binary level.

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| AND | 0x16 | 3 | Bitwise AND: `a & b` |
| OR | 0x17 | 3 | Bitwise OR: `a \| b` |
| XOR | 0x18 | 3 | Bitwise XOR: `a ^ b` |
| NOT | 0x19 | 3 | Bitwise NOT: `~a` |
| BYTE | 0x1A | 3 | Get the i-th byte from x |
| SHL | 0x1B | 3 | Shift left: `val << shift` |
| SHR | 0x1C | 3 | Logical shift right: `val >> shift` |
| SAR | 0x1D | 3 | Arithmetic shift right (preserves sign) |

---

## 🔐 4. CRYPTOGRAPHIC OPERATIONS

Opcodes for cryptographic hash functions.

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| KECCAK256 (SHA3) | 0x20 | 30* | Keccak-256 hash of memory region (dynamic gas) |

---

## 🌍 5. ENVIRONMENTAL INFORMATION

These opcodes access information about the current execution context, transaction, and contract.

### 5.1 Transaction Context

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| ADDRESS | 0x30 | 2 | Address of the currently executing contract |
| ORIGIN | 0x32 | 2 | Original transaction sender address (tx.origin) |
| CALLER | 0x33 | 2 | Direct caller address (msg.sender) |
| CALLVALUE | 0x34 | 2 | Wei sent with the call (msg.value) |
| GASPRICE | 0x3A | 2 | Transaction gas price in wei |


### 5.2 Call Data

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| CALLDATALOAD | 0x35 | 3 | Load 32 bytes from calldata |
| CALLDATASIZE | 0x36 | 2 | Size of calldata in bytes |
| CALLDATACOPY | 0x37 | 3* | Copy calldata to memory |

### 5.3 Code Access

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| CODESIZE | 0x38 | 2 | Size of current contract's code |
| CODECOPY | 0x39 | 3* | Copy code to memory |
| EXTCODESIZE | 0x3B | 100* | Size of code at another address (warm/cold access) |
| EXTCODECOPY | 0x3C | 100* | Copy external code to memory |
| EXTCODEHASH | 0x3F | 100* | Hash of code at another address |

### 5.4 Return Data

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| RETURNDATASIZE | 0x3D | 2 | Size of return data from last call |
| RETURNDATACOPY | 0x3E | 3* | Copy return data to memory |

### 5.5 Account Information

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| BALANCE | 0x31 | 100* | Balance of address in wei (warm/cold) |
| SELFBALANCE | 0x47 | 5 | Balance of current contract |

---

## 📦 6. BLOCK INFORMATION

These opcodes access data about the current block.

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| BLOCKHASH | 0x40 | 20 | Hash of a block (last 256 blocks only) |
| COINBASE | 0x41 | 2 | Block proposer/miner address |
| TIMESTAMP | 0x42 | 2 | Block Unix timestamp |
| NUMBER | 0x43 | 2 | Current block number |
| PREVRANDAO | 0x44 | 2 | Random beacon (formerly DIFFICULTY) |
| GASLIMIT | 0x45 | 2 | Block gas limit |
| CHAINID | 0x46 | 2 | Chain ID (EIP-155) |
| BASEFEE | 0x48 | 2 | Block base fee (EIP-1559) |
| BLOBHASH | 0x49 | 3 | Versioned blob hash (EIP-4844) |
| BLOBBASEFEE | 0x4A | 2 | Blob base fee for the block (EIP-7516) |

---

## 📚 7. STACK OPERATIONS

These opcodes manage the EVM stack, which is used for storing temporary values during execution.

### 7.1 Basic Stack

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| POP | 0x50 | 2 | Remove item from top of stack |

### 7.2 PUSH Operations (Push Constants to Stack)

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| PUSH0 | 0x5F | 2 | Push constant 0 |
| PUSH1 | 0x60 | 3 | Push 1-byte value |
| PUSH2 | 0x61 | 3 | Push 2-byte value |
| PUSH3 | 0x62 | 3 | Push 3-byte value |
| PUSH4 | 0x63 | 3 | Push 4-byte value |
| PUSH5 | 0x64 | 3 | Push 5-byte value |
| PUSH6 | 0x65 | 3 | Push 6-byte value |
| PUSH7 | 0x66 | 3 | Push 7-byte value |
| PUSH8 | 0x67 | 3 | Push 8-byte value |
| PUSH9 | 0x68 | 3 | Push 9-byte value |
| PUSH10 | 0x69 | 3 | Push 10-byte value |
| PUSH11 | 0x6A | 3 | Push 11-byte value |
| PUSH12 | 0x6B | 3 | Push 12-byte value |
| PUSH13 | 0x6C | 3 | Push 13-byte value |
| PUSH14 | 0x6D | 3 | Push 14-byte value |
| PUSH15 | 0x6E | 3 | Push 15-byte value |
| PUSH16 | 0x6F | 3 | Push 16-byte value |
| PUSH17 | 0x70 | 3 | Push 17-byte value |
| PUSH18 | 0x71 | 3 | Push 18-byte value |
| PUSH19 | 0x72 | 3 | Push 19-byte value |
| PUSH20 | 0x73 | 3 | Push 20-byte value |
| PUSH21 | 0x74 | 3 | Push 21-byte value |
| PUSH22 | 0x75 | 3 | Push 22-byte value |
| PUSH23 | 0x76 | 3 | Push 23-byte value |
| PUSH24 | 0x77 | 3 | Push 24-byte value |
| PUSH25 | 0x78 | 3 | Push 25-byte value |
| PUSH26 | 0x79 | 3 | Push 26-byte value |
| PUSH27 | 0x7A | 3 | Push 27-byte value |
| PUSH28 | 0x7B | 3 | Push 28-byte value |
| PUSH29 | 0x7C | 3 | Push 29-byte value |
| PUSH30 | 0x7D | 3 | Push 30-byte value |
| PUSH31 | 0x7E | 3 | Push 31-byte value |
| PUSH32 | 0x7F | 3 | Push 32-byte value (full word) |

### 7.3 DUP Operations (Duplicate Stack Items)

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| DUP1 | 0x80 | 3 | Duplicate 1st stack item |
| DUP2 | 0x81 | 3 | Duplicate 2nd stack item |
| DUP3 | 0x82 | 3 | Duplicate 3rd stack item |
| DUP4 | 0x83 | 3 | Duplicate 4th stack item |
| DUP5 | 0x84 | 3 | Duplicate 5th stack item |
| DUP6 | 0x85 | 3 | Duplicate 6th stack item |
| DUP7 | 0x86 | 3 | Duplicate 7th stack item |
| DUP8 | 0x87 | 3 | Duplicate 8th stack item |
| DUP9 | 0x88 | 3 | Duplicate 9th stack item |
| DUP10 | 0x89 | 3 | Duplicate 10th stack item |
| DUP11 | 0x8A | 3 | Duplicate 11th stack item |
| DUP12 | 0x8B | 3 | Duplicate 12th stack item |
| DUP13 | 0x8C | 3 | Duplicate 13th stack item |
| DUP14 | 0x8D | 3 | Duplicate 14th stack item |
| DUP15 | 0x8E | 3 | Duplicate 15th stack item |
| DUP16 | 0x8F | 3 | Duplicate 16th stack item |

### 7.4 SWAP Operations (Swap Stack Items)

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| SWAP1 | 0x90 | 3 | Swap top with 2nd item |
| SWAP2 | 0x91 | 3 | Swap top with 3rd item |
| SWAP3 | 0x92 | 3 | Swap top with 4th item |
| SWAP4 | 0x93 | 3 | Swap top with 5th item |
| SWAP5 | 0x94 | 3 | Swap top with 6th item |
| SWAP6 | 0x95 | 3 | Swap top with 7th item |
| SWAP7 | 0x96 | 3 | Swap top with 8th item |
| SWAP8 | 0x97 | 3 | Swap top with 9th item |
| SWAP9 | 0x98 | 3 | Swap top with 10th item |
| SWAP10 | 0x99 | 3 | Swap top with 11th item |
| SWAP11 | 0x9A | 3 | Swap top with 12th item |
| SWAP12 | 0x9B | 3 | Swap top with 13th item |
| SWAP13 | 0x9C | 3 | Swap top with 14th item |
| SWAP14 | 0x9D | 3 | Swap top with 15th item |
| SWAP15 | 0x9E | 3 | Swap top with 16th item |
| SWAP16 | 0x9F | 3 | Swap top with 17th item |

---

## 💾 8. MEMORY OPERATIONS

These opcodes manipulate volatile memory during execution.

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| MLOAD | 0x51 | 3* | Load word (32 bytes) from memory |
| MSTORE | 0x52 | 3* | Store word (32 bytes) to memory |
| MSTORE8 | 0x53 | 3* | Store single byte to memory |
| MSIZE | 0x59 | 2 | Current active memory size in bytes |
| MCOPY | 0x5E | 3* | Copy memory region (EIP-5656) |

*Gas includes potential memory expansion cost

---

## 💽 9. STORAGE OPERATIONS

These opcodes handle persistent storage on the blockchain.

### 9.1 Permanent Storage

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| SLOAD | 0x54 | 100* | Load word from storage (warm/cold) |
| SSTORE | 0x55 | 100* | Store word to storage (complex gas) |

### 9.2 Transient Storage (EIP-1153)

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| TLOAD | 0x5C | 100 | Load from transient storage |
| TSTORE | 0x5D | 100 | Store to transient storage |

*Transient storage is cleared after each transaction

---

## 🔀 10. CONTROL FLOW OPERATIONS

These opcodes control the execution flow of the program.

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| JUMP | 0x56 | 8 | Unconditional jump to destination |
| JUMPI | 0x57 | 10 | Conditional jump if condition ≠ 0 |
| JUMPDEST | 0x5B | 1 | Mark valid jump destination |
| PC | 0x58 | 2 | Program counter before increment |
| GAS | 0x5A | 2 | Remaining gas |
| STOP | 0x00 | 0 | Halt execution (success) |

---

## 📝 11. LOGGING OPERATIONS (Events)

These opcodes emit events to blockchain logs.

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| LOG0 | 0xA0 | 375* | Log with no topics |
| LOG1 | 0xA1 | 750* | Log with 1 topic |
| LOG2 | 0xA2 | 1125* | Log with 2 topics |
| LOG3 | 0xA3 | 1500* | Log with 3 topics |
| LOG4 | 0xA4 | 1875* | Log with 4 topics |

*Base cost + 8 gas per byte of data + 375 per topic

---

## 📞 12. SYSTEM OPERATIONS / CALLS

### 12.1 Contract Creation

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| CREATE | 0xF0 | 32000* | Create new contract |
| CREATE2 | 0xF5 | 32000* | Create with deterministic address |

### 12.2 External Calls

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| CALL | 0xF1 | 100* | Call external contract |
| CALLCODE | 0xF2 | 100* | Call with code from another contract (deprecated) |
| DELEGATECALL | 0xF4 | 100* | Call preserving msg.sender & msg.value |
| STATICCALL | 0xFA | 100* | Read-only call (no state modification) |

### 12.3 Return & Revert

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| RETURN | 0xF3 | 0* | Return data and halt (success) |
| REVERT | 0xFD | 0* | Revert with data (refund gas) |

### 12.4 Self Destruct

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| SELFDESTRUCT | 0xFF | 5000* | Destroy contract, send ETH (deprecated behavior post-Cancun) |

---

## ❌ 13. INVALID / HALT OPERATIONS

| Opcode | Hex  | Gas | Description |
|--------|------|-----|-------------|
| INVALID | 0xFE | All | Designated invalid opcode (EIP-141) |

---

## 🚫 14. UNDEFINED / RESERVED OPCODES

Opcode ranges that are undefined and will cause an error if executed:

| Range | Status |
|-------|--------|
| 0x0C - 0x0F | Invalid (Reserved) |
| 0x1E - 0x1F | Invalid (Reserved) |
| 0x21 - 0x2F | Invalid (Reserved) |
| 0x4B - 0x4F | Invalid (Reserved) |
| 0xA5 - 0xEF | Invalid (Reserved) |
| 0xF6 - 0xF9 | Invalid (Reserved) |
| 0xFB - 0xFC | Invalid (Reserved) |

---

## 📊 SUMMARY STATISTICS

| Category | Number of Opcodes |
|----------|-------------------|
| Arithmetic | 11 |
| Comparison | 6 |
| Bitwise | 8 |
| Cryptographic | 1 |
| Environmental | 16 |
| Block Information | 10 |
| Stack (POP) | 1 |
| Stack (PUSH) | 33 |
| Stack (DUP) | 16 |
| Stack (SWAP) | 16 |
| Memory | 5 |
| Storage | 4 |
| Control Flow | 6 |
| Logging | 5 |
| System/Calls | 9 |
| Invalid/Halt | 1 |
| **TOTAL VALID** | **148** |

---

## 🔧 TECHNICAL NOTES

### Gas Cost Types:
1. **Static Gas** - Fixed cost (e.g., ADD = 3 gas)
2. **Dynamic Gas** - Variable cost based on:
   - Memory expansion
   - Data size
   - Cold/Warm access (EIP-2929)
   - Storage slot status

### Access Lists (EIP-2929):
- **Cold Access**: First time accessing an address/slot = more expensive
- **Warm Access**: Subsequent access = cheaper

### EIP References:
- EIP-1153: Transient Storage (TLOAD, TSTORE)
- EIP-2929: Gas cost increases for state access
- EIP-4844: Blob transactions (BLOBHASH, BLOBBASEFEE)
- EIP-5656: MCOPY instruction

---

## 📖 REFERENCES

- [evm.codes](https://www.evm.codes) - Interactive reference
- [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
- [EIPs](https://eips.ethereum.org/)
- [go-ethereum source](https://github.com/ethereum/go-ethereum)

---

*Document Version: 1.0*


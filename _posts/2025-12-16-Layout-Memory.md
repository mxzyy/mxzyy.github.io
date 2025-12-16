---
title: Solidity Layout Memory (Pre-EVM Warmup)
date: 2025-12-16 09:00 +0700
categories: [Blockchain, Solidity]
tags: [solidity, evm, memory, storage, stack, fmp]
author: mxzyy
---

## Memory in EVM

Memory in EVM is a temporary storage area that is **volatile** (cleared after transaction execution). Memory is linear and byte-addressable, but read/written in 32-byte (256-bit) chunks.

Memory is used for:
- Storing temporary data during execution
- Passing arguments to external calls
- Return values from functions
- Hashing operations (keccak256)

## EVM Memory Layout

Memory in EVM has a specific layout reserved by Solidity:

```
┌─────────────────────────────────────────────────────────┐
│  0x00 - 0x3f  │  Scratch Space (64 bytes)               │
├─────────────────────────────────────────────────────────┤
│  0x40 - 0x5f  │  Free Memory Pointer (32 bytes)         │
├─────────────────────────────────────────────────────────┤
│  0x60 - 0x7f  │  Zero Slot (32 bytes)                   │
├─────────────────────────────────────────────────────────┤
│  0x80 - ...   │  Free Memory (starts here)              │
└─────────────────────────────────────────────────────────┘
```

### 1. Scratch Space (0x00 - 0x3f)

Scratch space is the first 64 bytes used for temporary operations. Commonly used for:
- Hashing (keccak256 requires data in memory)
- Inline assembly operations
- Temporary storage for intermediate operations

```solidity
// Example of scratch space usage in assembly
assembly {
    mstore(0x00, value1)  // Store in scratch space
    mstore(0x20, value2)  // Store in scratch space (offset 32 bytes)
    result := keccak256(0x00, 0x40)  // Hash 64 bytes
}
```

### 2. Free Memory Pointer (0x40 - 0x5f)

**Free Memory Pointer (FMP)** is a pointer that points to the next free memory location. This is a crucial concept in Solidity memory management.

- Location: `0x40` (64 in decimal)
- Initial value: `0x80` (128 in decimal)
- Solidity automatically updates FMP every time memory is allocated

```solidity
// Reading Free Memory Pointer
assembly {
    let fmp := mload(0x40)  // Read FMP value
    // fmp now contains the next free memory address
}
```

#### How FMP Works

```
Initial: FMP = 0x80

1. Allocate 32-byte array:
   - Data written at 0x80
   - FMP updated to 0xa0 (0x80 + 0x20)

2. Allocate 64-byte struct:
   - Data written at 0xa0
   - FMP updated to 0xe0 (0xa0 + 0x40)
```

```solidity
// Example of manual memory allocation
assembly {
    // Read current FMP
    let ptr := mload(0x40)

    // Allocate 32 bytes
    mstore(ptr, someValue)

    // Update FMP to new location
    mstore(0x40, add(ptr, 0x20))
}
```

### 3. Zero Slot (0x60 - 0x7f)

Zero slot is 32 bytes that always contains zero. Used as:
- Initial value for dynamic memory arrays
- Reference for empty values

## Stack, Memory, and Storage

EVM has three main storage areas with different characteristics:

### Stack

Stack is the fastest and cheapest (gas-wise) storage area.

| Characteristic | Value |
|----------------|-------|
| Item size | 256 bits (32 bytes) |
| Max depth | 1024 items |
| Access | LIFO (Last In, First Out) |
| Gas cost | Very low |
| Lifetime | During function execution |

```
Stack Operations:
┌─────────────────────────────────────────┐
│  PUSH1 0x05    │  Push 5 to stack       │
│  PUSH1 0x03    │  Push 3 to stack       │
│  ADD           │  Pop 2, push result (8)│
└─────────────────────────────────────────┘

Stack state:
Before ADD:  [5, 3, ...]  (3 on top)
After ADD:   [8, ...]     (8 on top)
```

### Memory

Memory is a volatile area more flexible than stack.

| Characteristic | Value |
|----------------|-------|
| Size | Expandable (gas cost increases) |
| Access | Random access (byte-addressable) |
| Read/Write | 32 bytes at once (MLOAD/MSTORE) |
| Gas cost | Cheap for small sizes, expensive for large |
| Lifetime | During transaction execution |

```solidity
// Memory operations in Solidity
function memoryExample() public pure returns (bytes32) {
    bytes32 data;

    assembly {
        // MSTORE: store 32 bytes to memory
        mstore(0x80, 0x1234)

        // MLOAD: read 32 bytes from memory
        data := mload(0x80)

        // MSTORE8: store 1 byte to memory
        mstore8(0x80, 0xff)
    }

    return data;
}
```

**Memory Expansion Cost:**

Memory expansion is not free. Gas cost for memory is calculated with the formula:

```
memory_cost = (memory_size_word ^ 2) / 512 + (3 * memory_size_word)
```

The larger the memory used, the more expensive it becomes (quadratic growth).

### Storage

Storage is persistent storage saved on the blockchain.

| Characteristic | Value |
|----------------|-------|
| Slot size | 256 bits (32 bytes) |
| Access | Key-value (slot number) |
| Gas cost | Very expensive |
| Lifetime | Permanent (until modified) |

```solidity
contract StorageExample {
    // Slot 0
    uint256 public value1;

    // Slot 1
    uint256 public value2;

    // Slot 2 (packed: address = 20 bytes, uint96 = 12 bytes)
    address public owner;      // 20 bytes
    uint96 public smallValue;  // 12 bytes

    function storageOps() public {
        assembly {
            // SSTORE: store to storage slot 0
            sstore(0, 100)

            // SLOAD: read from storage slot 0
            let val := sload(0)
        }
    }
}
```

**Storage Layout:**

```
┌─────────────────────────────────────────────────────────────────────┐
│ Slot 0: value1                                                      │
│ [0x0000000000000000000000000000000000000000000000000000000000000000]│
├─────────────────────────────────────────────────────────────────────┤
│ Slot 1: value2                                                      │
│ [0x0000000000000000000000000000000000000000000000000000000000000000]│
├─────────────────────────────────────────────────────────────────────┤
│ Slot 2: owner (20 bytes) + smallValue (12 bytes) - PACKED           │
│ [0x000000000000000000000000AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA]│
│                            └─────────── owner ───────────┘          │
└─────────────────────────────────────────────────────────────────────┘
```

## Comprehensive Example

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MemoryLayoutDemo {
    // Storage variables
    uint256 public storedValue;  // Slot 0

    function demonstrateAll(uint256 input) public returns (uint256) {
        uint256 result;

        assembly {
            // ========== STACK ==========
            // Push values to stack
            let a := 10          // Stack: [10]
            let b := 20          // Stack: [10, 20]
            let sum := add(a, b) // Stack: [30] (ADD result)

            // ========== MEMORY ==========
            // Read Free Memory Pointer
            let fmp := mload(0x40)  // fmp = 0x80 (initial)

            // Store data in memory (starting from FMP)
            mstore(fmp, input)           // Store input at 0x80
            mstore(add(fmp, 0x20), sum)  // Store sum at 0xa0

            // Update FMP (we used 64 bytes)
            mstore(0x40, add(fmp, 0x40))

            // Use scratch space for hashing
            mstore(0x00, input)
            mstore(0x20, sum)
            let hash := keccak256(0x00, 0x40)

            // ========== STORAGE ==========
            // Store hash to storage slot 0
            sstore(0, hash)

            // Read back from storage
            result := sload(0)
        }

        return result;
    }

    function readFMP() public pure returns (uint256 fmp) {
        assembly {
            fmp := mload(0x40)
        }
    }

    function allocateMemory(uint256 size) public pure returns (uint256 ptr) {
        assembly {
            // Read current FMP
            ptr := mload(0x40)

            // Update FMP
            mstore(0x40, add(ptr, size))
        }
    }
}
```

## Gas Cost Comparison

| Operation | Gas Cost (approximate) |
|-----------|------------------------|
| Stack PUSH | 3 |
| Stack POP | 2 |
| MLOAD | 3 |
| MSTORE | 3 |
| SLOAD (cold) | 2100 |
| SLOAD (warm) | 100 |
| SSTORE (cold, 0→non-0) | 22100 |
| SSTORE (warm) | 100 |

## Key Takeaways

1. **Stack** is fastest and cheapest, but limited (1024 depth, LIFO only)
2. **Memory** is flexible for temporary data, cost increases quadratically
3. **Storage** is persistent but very expensive, use wisely
4. **FMP** at `0x40` is the key to memory management in Solidity
5. **Scratch space** (0x00-0x3f) for temporary operations without modifying FMP
6. Memory always starts from `0x80` because 0x00-0x7f is reserved

Understanding memory layout is fundamental for:
- Gas optimization
- Writing secure inline assembly
- Debugging low-level issues
- Understanding how Solidity manages data

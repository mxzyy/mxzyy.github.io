---
title: EVM Storage Slot
date: 2026-01-01 2:30 +0700
categories: [Blockchain, Ethereum, Solidity]
tags: [evm, storage, solidity, smart-contracts]
author: mxzyy
---

## Introduction

Storage slots are the foundation of how data persists on the Ethereum blockchain. Every state variable in your smart contract lives in storage, and understanding how the EVM organizes this data is crucial for gas optimization, security auditing, and low-level contract development.

When you deploy a contract, every piece of persistent data—balances, mappings, arrays—gets assigned to specific 32-byte storage locations called "slots." The EVM uses a key-value store where each key is a 256-bit slot number and each value is a 256-bit word. This seemingly simple structure enables incredibly complex data organization through clever use of hashing.

This guide will take you from the basics of slot assignment through advanced topics like nested mappings and dynamic arrays, with practical examples you can apply immediately.

---

## What is a Slot?

Think of EVM storage as an enormous filing cabinet with 2²⁵⁶ drawers. Each drawer (slot) can hold exactly 32 bytes of data. When you declare a state variable, the compiler assigns it a drawer number starting from 0.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        EVM Storage Model                            │
├─────────────────────────────────────────────────────────────────────┤
│  Slot 0   │ [32 bytes of data]                                      │
│  Slot 1   │ [32 bytes of data]                                      │
│  Slot 2   │ [32 bytes of data]                                      │
│    ...    │       ...                                               │
│  Slot N   │ [32 bytes of data]                                      │
│    ...    │       ...                                               │
│  Slot 2²⁵⁶-1 │ [32 bytes of data]                                   │
└─────────────────────────────────────────────────────────────────────┘
```

**Key characteristics:**
- Each slot holds exactly **32 bytes** (256 bits)
- Slot numbers range from **0 to 2²⁵⁶ - 1**
- Storage is **sparse**—you only pay for slots you actually use
- Reading/writing storage is **expensive** (SLOAD/SSTORE operations)

---

## Storage Layout Basics

State variables are assigned to slots **sequentially** in the order they're declared, starting from slot 0.

```
contract BasicLayout {
    uint256 public a;      // Slot 0
    uint256 public b;      // Slot 1
    uint256 public c;      // Slot 2
}
```

```
┌──────────────────────────────────────────────────────────────────────┐
│ Slot 0: a                                                            │
│ [0x0000000000000000000000000000000000000000000000000000000000000000] │
├──────────────────────────────────────────────────────────────────────┤
│ Slot 1: b                                                            │
│ [0x0000000000000000000000000000000000000000000000000000000000000000] │
├──────────────────────────────────────────────────────────────────────┤
│ Slot 2: c                                                            │
│ [0x0000000000000000000000000000000000000000000000000000000000000000] │
└──────────────────────────────────────────────────────────────────────┘
```

The assignment is straightforward: first declared variable gets slot 0, second gets slot 1, and so on. But what happens when variables are smaller than 32 bytes?

---

## Data Types and Byte Sizes

Not all Solidity types need the full 32 bytes. Here's a reference table:

| Type | Size | Description |
|------|------|-------------|
| `bool` | 1 byte | True (1) or False (0) |
| `uint8` / `int8` | 1 byte | 8-bit integer |
| `uint16` / `int16` | 2 bytes | 16-bit integer |
| `uint32` / `int32` | 4 bytes | 32-bit integer |
| `uint64` / `int64` | 8 bytes | 64-bit integer |
| `uint128` / `int128` | 16 bytes | 128-bit integer |
| `uint256` / `int256` | 32 bytes | 256-bit integer (full slot) |
| `address` | 20 bytes | Ethereum address |
| `bytes1` to `bytes32` | 1-32 bytes | Fixed-size byte arrays |
| `bytes` | 32 bytes* | Dynamic byte array (pointer) |
| `string` | 32 bytes* | Dynamic string (pointer) |
| `mapping` | 32 bytes | Only reserves slot for base |
| `array[]` | 32 bytes | Dynamic array (stores length) |
| `array[n]` | n × element size | Fixed array (inline storage) |

*Dynamic types store length/pointer in the base slot; actual data is elsewhere.

---

## Variable Packing

The Solidity compiler **packs** multiple smaller variables into a single slot when possible. This is a critical gas optimization technique.

### How Packing Works

Variables are packed from **right to left** (low-order bytes first) within a slot:

```
contract PackedVariables {
    uint128 public a;  // Slot 0 (lower 16 bytes)
    uint128 public b;  // Slot 0 (upper 16 bytes)
    uint256 public c;  // Slot 1 (needs full slot)
}
```

```
┌──────────────────────────────────────────────────────────────────────┐
│ Slot 0: a (16 bytes) + b (16 bytes) = 32 bytes PACKED                │
│ [BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA]   │
│  └────────── b ──────────┘└────────── a ──────────┘                  │
├──────────────────────────────────────────────────────────────────────┤
│ Slot 1: c (32 bytes)                                                 │
│ [CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC]   │
└──────────────────────────────────────────────────────────────────────┘
```

### Practical Packing Example

```
contract EfficientStorage {
    // GOOD: Variables packed efficiently (2 slots total)
    address public owner;     // 20 bytes ─┐
    uint96 public balance;    // 12 bytes ─┘ Slot 0 (32 bytes)
    bool public isActive;     // 1 byte  ─┐
    uint8 public level;       // 1 byte   │
    uint16 public score;      // 2 bytes  │ Slot 1 (packed)
    uint224 public data;      // 28 bytes ┘
}

contract InefficientStorage {
    // BAD: Poor ordering wastes slots (6 slots total)
    bool public isActive;     // 1 byte  → Slot 0 (31 bytes wasted!)
    uint256 public data;      // 32 bytes → Slot 1
    uint8 public level;       // 1 byte  → Slot 2 (31 bytes wasted!)
    address public owner;     // 20 bytes → Slot 3 (12 bytes wasted!)
    uint16 public score;      // 2 bytes → Slot 4 (30 bytes wasted!)
    uint96 public balance;    // 12 bytes → Slot 5 (20 bytes wasted!)
}
```

> **Tip:** Group smaller variables together and order them by size to maximize packing efficiency. The difference can be significant in contracts with many state variables.

---

## Dynamic Arrays

Dynamic arrays have a two-part storage mechanism: the **length** is stored at the declared slot, while the **actual elements** are stored at a location derived from hashing.

### Storage Formula

```
Array length → Stored at slot p
Element[i]   → Stored at keccak256(p) + i
```

Where `p` is the slot number where the array is declared.

### Why Hashing?

Consider what would happen without hashing:

```
Without hashing (HYPOTHETICAL - NOT HOW IT WORKS):
┌─────────────────────────────────────────────────────────────────────┐
│ Slot 0: array length                                                │
│ Slot 1: array[0]                                                    │
│ Slot 2: array[1]                                                    │
│ Slot 3: nextVariable  ← COLLISION if array grows!                   │
└─────────────────────────────────────────────────────────────────────┘
```

The array could overwrite other variables! Hashing solves this by placing array data at a pseudo-random location far from sequential slots:

```
With hashing (ACTUAL IMPLEMENTATION):
┌─────────────────────────────────────────────────────────────────────┐
│ Slot 0: array length = 3                                            │
│ Slot 1: nextVariable (safe!)                                        │
│ ...                                                                 │
│ Slot keccak256(0): array[0]                                         │
│ Slot keccak256(0)+1: array[1]                                       │
│ Slot keccak256(0)+2: array[2]                                       │
└─────────────────────────────────────────────────────────────────────┘
```

### Step-by-Step Example

```
contract DynamicArrayStorage {
    uint256[] public numbers;  // Slot 0 (stores length)
    uint256 public value;      // Slot 1 (safe from array collision)

    function addNumbers() public {
        numbers.push(100);
        numbers.push(200);
        numbers.push(300);
    }
}
```

**After calling `addNumbers()`:**

```
┌──────────────────────────────────────────────────────────────────────┐
│ Slot 0: numbers.length = 3                                           │
│ [0x0000000000000000000000000000000000000000000000000000000000000003] │
├──────────────────────────────────────────────────────────────────────┤
│ Slot 1: value = 0                                                    │
│ [0x0000000000000000000000000000000000000000000000000000000000000000] │
├──────────────────────────────────────────────────────────────────────┤
│                           ...                                        │
├──────────────────────────────────────────────────────────────────────┤
│ Slot keccak256(0): numbers[0] = 100                                  │
│ [0x0000000000000000000000000000000000000000000000000000000000000064] │
├──────────────────────────────────────────────────────────────────────┤
│ Slot keccak256(0)+1: numbers[1] = 200                                │
│ [0x00000000000000000000000000000000000000000000000000000000000000c8] │
├──────────────────────────────────────────────────────────────────────┤
│ Slot keccak256(0)+2: numbers[2] = 300                                │
│ [0x000000000000000000000000000000000000000000000000000000000000012c] │
└──────────────────────────────────────────────────────────────────────┘

Where keccak256(0) = 0x290decd9548b62a8d60345a988386fc84ba6bc95484008f6362f93160ef3e563
```

### Push Operation Walkthrough

When you call `push()` on a dynamic array:

```
Initial State:
┌────────────────────────────────────────┐
│ Slot 0: length = 0                     │
└────────────────────────────────────────┘

After push(100):
┌────────────────────────────────────────┐
│ Slot 0: length = 1                     │
│ Slot keccak256(0)+0: 100               │
└────────────────────────────────────────┘

After push(200):
┌────────────────────────────────────────┐
│ Slot 0: length = 2                     │
│ Slot keccak256(0)+0: 100               │
│ Slot keccak256(0)+1: 200               │
└────────────────────────────────────────┘

After push(300):
┌────────────────────────────────────────┐
│ Slot 0: length = 3                     │
│ Slot keccak256(0)+0: 100               │
│ Slot keccak256(0)+1: 200               │
│ Slot keccak256(0)+2: 300               │
└────────────────────────────────────────┘
```

---

## Mappings

Mappings use a different hashing formula than arrays. Since mappings don't have a length (they're conceptually infinite), nothing is stored at the declared slot—it's only used as a "base" for computing actual storage locations.

### Storage Formula

```
mapping[key] → Stored at keccak256(key . p)
```

Where:
- `key` is the mapping key (padded to 32 bytes)
- `p` is the slot number where the mapping is declared
- `.` represents concatenation

### Why This Design?

Mappings only allocate storage for keys that actually exist. The hash ensures:

1. **No collisions** between different keys
2. **No collisions** with other variables
3. **Sparse storage**—unused keys cost nothing

### Mapping Example

```
contract MappingStorage {
    mapping(address => uint256) public balances;  // Slot 0
    uint256 public totalSupply;                   // Slot 1

    function setBalances() public {
        balances[0xAAA...AAA] = 1000;
        balances[0xBBB...BBB] = 2000;
    }
}
```

**Slot calculation for `balances[0xAAA...AAA]`:**

```
// Step 1: Pad the key to 32 bytes
key = 0x000000000000000000000000AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

// Step 2: Pad the slot number to 32 bytes
slot = 0x0000000000000000000000000000000000000000000000000000000000000000

// Step 3: Concatenate and hash
storage_location = keccak256(key . slot)
                 = keccak256(0x000000000000000000000000AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
                             0000000000000000000000000000000000000000000000000000000000000000)
```

```
┌──────────────────────────────────────────────────────────────────────┐
│ Slot 0: (empty - mapping base, nothing stored here)                  │
│ [0x0000000000000000000000000000000000000000000000000000000000000000] │
├──────────────────────────────────────────────────────────────────────┤
│ Slot 1: totalSupply                                                  │
│ [0x0000000000000000000000000000000000000000000000000000000000000000] │
├──────────────────────────────────────────────────────────────────────┤
│                              ...                                     │
├──────────────────────────────────────────────────────────────────────┤
│ Slot keccak256(0xAAA...|0): balances[0xAAA...] = 1000                │
│ [0x00000000000000000000000000000000000000000000000000000000000003e8] │
├──────────────────────────────────────────────────────────────────────┤
│ Slot keccak256(0xBBB...|0): balances[0xBBB...] = 2000                │
│ [0x00000000000000000000000000000000000000000000000000000000000007d0] │
└──────────────────────────────────────────────────────────────────────┘
```

> **Important:** The base slot (Slot 0 in this case) is **reserved** but **empty**. It's only used in the hash calculation. This is different from arrays, which store their length at the base slot.

---

## Nested Structures

When you combine mappings, arrays, and structs, the storage calculations become more complex. Let's break down each case.

### Nested Mappings

For nested mappings like `mapping(address => mapping(uint256 => bool))`, you apply the hashing formula recursively.

```
contract NestedMapping {
    // mapping(address => mapping(uint256 => bool))
    mapping(address => mapping(uint256 => bool)) public permissions;  // Slot 0
}
```

**Formula for `permissions[addr][id]`:**

```
Step 1: Calculate slot for permissions[addr]
        tempSlot = keccak256(addr . 0)

Step 2: Calculate final slot for permissions[addr][id]
        finalSlot = keccak256(id . tempSlot)
```

> **Important Clarification:** `tempSlot` is NOT stored anywhere—it's just an intermediate value in the calculation. The EVM computes this on-the-fly when you access the nested value.

**Concrete Example:**

```
permissions[0xAAA...AAA][42] = true;
```

```
Step 1: tempSlot = keccak256(
    0x000000000000000000000000AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA  // addr (padded)
    0000000000000000000000000000000000000000000000000000000000000000    // slot 0
)
= 0x8c6... (some hash)

Step 2: finalSlot = keccak256(
    0x000000000000000000000000000000000000000000000000000000000000002a  // 42 (padded)
    8c6...                                                             // tempSlot
)
= 0xf3a... (final storage location)

Storage:
┌──────────────────────────────────────────────────────────────────────┐
│ Slot 0: (empty - outer mapping base)                                 │
├──────────────────────────────────────────────────────────────────────┤
│ Slot 0x8c6...: (empty - inner mapping base, tempSlot)                │
│                 This slot exists conceptually but stores nothing!    │
├──────────────────────────────────────────────────────────────────────┤
│ Slot 0xf3a...: permissions[0xAAA...][42] = true                      │
│ [0x0000000000000000000000000000000000000000000000000000000000000001] │
└──────────────────────────────────────────────────────────────────────┘
```

### Mapping to Array

```
contract MappingToArray {
    mapping(address => uint256[]) public userScores;  // Slot 0
}
```

**Formula for `userScores[addr][i]`:**

```
Step 1: Calculate array base slot
        arraySlot = keccak256(addr . 0)
        // This slot stores the array LENGTH

Step 2: Calculate element location
        elementSlot = keccak256(arraySlot) + i
```

```
userScores[0xAAA...AAA].push(100);
userScores[0xAAA...AAA].push(200);
```

```
Storage Layout:
┌─────────────────────────────────────────────────────────────────────┐
│ Slot 0: (empty - mapping base)                                      │
├─────────────────────────────────────────────────────────────────────┤
│ Slot keccak256(0xAAA...|0): userScores[0xAAA...].length = 2         │
├─────────────────────────────────────────────────────────────────────┤
│ Slot keccak256(keccak256(0xAAA...|0))+0: userScores[0xAAA...][0]=100│
├─────────────────────────────────────────────────────────────────────┤
│ Slot keccak256(keccak256(0xAAA...|0))+1: userScores[0xAAA...][1]=200│
└─────────────────────────────────────────────────────────────────────┘
```

### Array of Structs

```
contract ArrayOfStructs {
    struct User {
        address addr;     // 20 bytes ─┐
        uint96 balance;   // 12 bytes ─┘ Packed in 1 slot
        uint256 score;    // 32 bytes → 1 slot
    }
    // Each User takes 2 slots (struct size = 2)

    User[] public users;  // Slot 0
}
```

**Formula for `users[i].field`:**

```
Base data location = keccak256(0)
Struct size = 2 slots

users[i].addr    → keccak256(0) + (i * 2) + 0  (lower 20 bytes)
users[i].balance → keccak256(0) + (i * 2) + 0  (upper 12 bytes)
users[i].score   → keccak256(0) + (i * 2) + 1
```

```
After adding 2 users:
┌─────────────────────────────────────────────────────────────────────┐
│ Slot 0: users.length = 2                                            │
├─────────────────────────────────────────────────────────────────────┤
│ Slot keccak256(0)+0: users[0].addr + users[0].balance (packed)      │
│ Slot keccak256(0)+1: users[0].score                                 │
├─────────────────────────────────────────────────────────────────────┤
│ Slot keccak256(0)+2: users[1].addr + users[1].balance (packed)      │
│ Slot keccak256(0)+3: users[1].score                                 │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Slot Indexing: Why 0 to 2²⁵⁶-1?

The EVM uses **256-bit keys** for storage slots, giving us 2²⁵⁶ possible slots. This astronomical number (~10⁷⁷) ensures:

1. **Hash collision resistance**: With keccak256 producing 256-bit hashes, the probability of two different inputs producing the same storage location is negligible.

2. **Sparse storage model**: We can use any slot without worrying about "running out" of space. Only slots that actually contain data cost gas.

3. **Deterministic computation**: Given the same inputs, the storage location is always the same, enabling predictable state access.

```
Total slots: 2²⁵⁶ = 115,792,089,237,316,195,423,570,985,008,687,907,853,269,984,665,640,564,039,457,584,007,913,129,639,936

For perspective:
- Atoms in observable universe: ~10⁸⁰
- 2²⁵⁶ ≈ 10⁷⁷

Even with billions of contracts each using millions of slots, collision probability remains negligible.
```

---

## Storage Operations: SLOAD and SSTORE

The EVM provides two opcodes for storage access:

### SLOAD (Load from Storage)

```
Opcode: 0x54
Input: slot (from stack)
Output: value (to stack)
```

| Access Type | Gas Cost |
|-------------|----------|
| Cold (first access) | 2100 gas |
| Warm (subsequent) | 100 gas |

### SSTORE (Store to Storage)

```
Opcode: 0x55
Input: slot, value (from stack)
Output: none
```

| Scenario | Gas Cost |
|----------|----------|
| Zero → Non-zero (cold) | 22,100 gas |
| Non-zero → Non-zero (cold) | 5,000 gas |
| Non-zero → Zero (cold) | 5,000 gas + 4,800 refund |
| Warm access | 100 gas (base) |

### Assembly Example

```
contract StorageOps {
    uint256 public value;  // Slot 0

    function readSlot(uint256 slot) public view returns (uint256 result) {
        assembly {
            result := sload(slot)
        }
    }

    function writeSlot(uint256 slot, uint256 data) public {
        assembly {
            sstore(slot, data)
        }
    }

    // Read any mapping slot
    function readMappingSlot(
        uint256 mappingSlot,
        address key
    ) public view returns (uint256 result) {
        assembly {
            // Store key and slot in memory for hashing
            mstore(0x00, key)
            mstore(0x20, mappingSlot)

            // Compute storage location
            let location := keccak256(0x00, 0x40)

            // Load value
            result := sload(location)
        }
    }
}
```

---

## Practical Examples

### Example 1: ERC20 Token Storage

```
contract SimpleERC20 {
    string public name;                           // Slot 0
    string public symbol;                         // Slot 1
    uint8 public decimals;                        // Slot 2
    uint256 public totalSupply;                   // Slot 3
    mapping(address => uint256) public balanceOf; // Slot 4
    mapping(address => mapping(address => uint256)) public allowance; // Slot 5
}
```

```
Storage Layout:
┌─────────────────────────────────────────────────────────────────────┐
│ Slot 0: name (short string or pointer)                              │
│ Slot 1: symbol (short string or pointer)                            │
│ Slot 2: decimals (1 byte, 31 bytes unused)                          │
│ Slot 3: totalSupply                                                 │
│ Slot 4: balanceOf mapping base (empty)                              │
│ Slot 5: allowance mapping base (empty)                              │
├─────────────────────────────────────────────────────────────────────┤
│ balanceOf[addr] → keccak256(addr | 4)                               │
│ allowance[owner][spender] → keccak256(spender | keccak256(owner|5)) │
└─────────────────────────────────────────────────────────────────────┘
```

### Example 2: NFT Ownership Tracking

```
contract SimpleNFT {
    mapping(uint256 => address) public ownerOf;        // Slot 0
    mapping(address => uint256) public balanceOf;      // Slot 1
    mapping(uint256 => address) public getApproved;    // Slot 2
    mapping(address => mapping(address => bool)) public isApprovedForAll; // Slot 3
}
```

**Query: Who owns token #42?**

```
// Storage location for ownerOf[42]:
slot = keccak256(
    0x000000000000000000000000000000000000000000000000000000000000002a  // 42
    0000000000000000000000000000000000000000000000000000000000000000    // slot 0
)
```

### Example 3: Reading Packed Variables

```
contract PackedReader {
    address public owner;     // Slot 0: bytes 0-19
    uint96 public balance;    // Slot 0: bytes 20-31

    function readPacked() public view returns (address, uint96) {
        address _owner;
        uint96 _balance;

        assembly {
            // Load entire slot 0
            let data := sload(0)

            // Extract owner (lower 20 bytes)
            _owner := and(data, 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF)

            // Extract balance (upper 12 bytes, shift right 160 bits)
            _balance := shr(160, data)
        }

        return (_owner, _balance);
    }
}
```

---

## Conclusion

### Key Takeaways

1. **Sequential Assignment**: State variables get slots 0, 1, 2... in declaration order.

2. **Packing Saves Gas**: Group smaller variables together. Order matters!

3. **Arrays Use Hashing**: Length at base slot, data at `keccak256(slot) + index`.

4. **Mappings Use Hashing**: Value at `keccak256(key | slot)`. Base slot stays empty.

5. **Nested Structures**: Apply formulas recursively. Intermediate slots (tempSlot) are computed, not stored.

6. **2²⁵⁶ Slots**: Ensures collision resistance for hash-based addressing.

7. **Gas Awareness**: SSTORE is expensive. Cold access costs more than warm. Zero→non-zero is most expensive.

### Best Practices

```
// DO: Group small variables for packing
address owner;      // 20 bytes ─┐
uint96 balance;     // 12 bytes ─┘ 1 slot

// DO: Use mappings for sparse data
mapping(uint256 => Data) items;  // Only pays for used keys

// DO: Consider storage layout in upgradeable contracts
// New variables must be added at the end

// DON'T: Interleave large and small variables
uint256 a;
bool b;      // Wastes 31 bytes
uint256 c;

// DON'T: Store data you can compute
// Bad: storing keccak256(x) when x is already stored
```

### Further Reading

- [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) - Formal specification
- [Solidity Documentation - Layout of State Variables](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html)
- [EIP-2929](https://eips.ethereum.org/EIPS/eip-2929) - Gas cost changes for state access
- [evm.codes](https://www.evm.codes) - Interactive opcode reference

---

*Understanding storage slots is fundamental for writing gas-efficient contracts, performing security audits, and building tools that interact with on-chain data. Master these concepts, and you'll have a significant advantage in EVM development.*

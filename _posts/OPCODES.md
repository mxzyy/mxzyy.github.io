# 📚 Klasifikasi Lengkap EVM Opcodes

## Referensi: https://www.evm.codes | Ethereum Yellow Paper

---

## 🔢 1. ARITHMETIC OPERATIONS (Operasi Aritmatika)

Opcodes untuk melakukan operasi matematika pada nilai 256-bit.

| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| ADD | 0x01 | 3 | Penjumlahan: `a + b` (mod 2²⁵⁶) |
| MUL | 0x02 | 5 | Perkalian: `a * b` (mod 2²⁵⁶) |
| SUB | 0x03 | 3 | Pengurangan: `a - b` (mod 2²⁵⁶) |
| DIV | 0x04 | 5 | Pembagian unsigned: `a / b` |
| SDIV | 0x05 | 5 | Pembagian signed (int256) |
| MOD | 0x06 | 5 | Modulus unsigned: `a % b` |
| SMOD | 0x07 | 5 | Modulus signed (int256) |
| ADDMOD | 0x08 | 8 | Penjumlahan modular: `(a + b) % N` |
| MULMOD | 0x09 | 8 | Perkalian modular: `(a * b) % N` |
| EXP | 0x0A | 10* | Eksponensial: `a ** b` (dynamic gas) |
| SIGNEXTEND | 0x0B | 5 | Sign extension dari (b+1) bytes ke 32 bytes |

---

## ⚖️ 2. COMPARISON OPERATIONS (Operasi Perbandingan)

Opcodes untuk membandingkan nilai dan menghasilkan boolean (0 atau 1).

| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| LT | 0x10 | 3 | Less than (unsigned): `a < b` |
| GT | 0x11 | 3 | Greater than (unsigned): `a > b` |
| SLT | 0x12 | 3 | Signed less than (int256) |
| SGT | 0x13 | 3 | Signed greater than (int256) |
| EQ | 0x14 | 3 | Equality: `a == b` |
| ISZERO | 0x15 | 3 | Check zero: `a == 0` |

---

## 🔣 3. BITWISE OPERATIONS (Operasi Bitwise)

Opcodes untuk manipulasi bit pada level binary.

| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| AND | 0x16 | 3 | Bitwise AND: `a & b` |
| OR | 0x17 | 3 | Bitwise OR: `a \| b` |
| XOR | 0x18 | 3 | Bitwise XOR: `a ^ b` |
| NOT | 0x19 | 3 | Bitwise NOT: `~a` |
| BYTE | 0x1A | 3 | Ambil byte ke-i dari x |
| SHL | 0x1B | 3 | Shift left: `val << shift` |
| SHR | 0x1C | 3 | Logical shift right: `val >> shift` |
| SAR | 0x1D | 3 | Arithmetic shift right (preserves sign) |

---

## 🔐 4. CRYPTOGRAPHIC OPERATIONS (Operasi Kriptografi)

Opcodes untuk fungsi hash kriptografi.

| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| KECCAK256 (SHA3) | 0x20 | 30* | Hash Keccak-256 dari memory region (dynamic gas) |

---

## 🌍 5. ENVIRONMENTAL INFORMATION (Informasi Lingkungan)

Opcodes untuk mengakses informasi tentang eksekusi saat ini, transaksi, dan contract.

### 5.1 Transaction Context
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| ADDRESS | 0x30 | 2 | Address contract yang sedang dieksekusi |
| ORIGIN | 0x32 | 2 | Address pengirim transaksi asal (tx.origin) |
| CALLER | 0x33 | 2 | Address pemanggil langsung (msg.sender) |
| CALLVALUE | 0x34 | 2 | Wei yang dikirim (msg.value) |
| GASPRICE | 0x3A | 2 | Gas price transaksi dalam wei |

### 5.2 Call Data
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| CALLDATALOAD | 0x35 | 3 | Load 32 bytes dari calldata |
| CALLDATASIZE | 0x36 | 2 | Ukuran calldata dalam bytes |
| CALLDATACOPY | 0x37 | 3* | Copy calldata ke memory |

### 5.3 Code Access
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| CODESIZE | 0x38 | 2 | Ukuran code contract saat ini |
| CODECOPY | 0x39 | 3* | Copy code ke memory |
| EXTCODESIZE | 0x3B | 100* | Ukuran code di address lain (warm/cold access) |
| EXTCODECOPY | 0x3C | 100* | Copy external code ke memory |
| EXTCODEHASH | 0x3F | 100* | Hash dari code di address lain |

### 5.4 Return Data
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| RETURNDATASIZE | 0x3D | 2 | Ukuran return data dari call terakhir |
| RETURNDATACOPY | 0x3E | 3* | Copy return data ke memory |

### 5.5 Account Information
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| BALANCE | 0x31 | 100* | Balance address dalam wei (warm/cold) |
| SELFBALANCE | 0x47 | 5 | Balance contract sendiri |

---

## 📦 6. BLOCK INFORMATION (Informasi Block)

Opcodes untuk mengakses data block saat ini.

| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| BLOCKHASH | 0x40 | 20 | Hash dari block (256 block terakhir) |
| COINBASE | 0x41 | 2 | Address block proposer/miner |
| TIMESTAMP | 0x42 | 2 | Unix timestamp block |
| NUMBER | 0x43 | 2 | Nomor block saat ini |
| PREVRANDAO | 0x44 | 2 | Random beacon (sebelumnya DIFFICULTY) |
| GASLIMIT | 0x45 | 2 | Gas limit block |
| CHAINID | 0x46 | 2 | Chain ID (EIP-155) |
| BASEFEE | 0x48 | 2 | Base fee block (EIP-1559) |
| BLOBHASH | 0x49 | 3 | Versioned hash blob (EIP-4844) |
| BLOBBASEFEE | 0x4A | 2 | Blob base fee block (EIP-7516) |

---

## 📚 7. STACK OPERATIONS (Operasi Stack)

### 7.1 Basic Stack
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| POP | 0x50 | 2 | Hapus item dari top stack |

### 7.2 PUSH Operations (Push Constant ke Stack)
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
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
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| DUP1 | 0x80 | 3 | Duplicate item ke-1 di stack |
| DUP2 | 0x81 | 3 | Duplicate item ke-2 di stack |
| DUP3 | 0x82 | 3 | Duplicate item ke-3 di stack |
| DUP4 | 0x83 | 3 | Duplicate item ke-4 di stack |
| DUP5 | 0x84 | 3 | Duplicate item ke-5 di stack |
| DUP6 | 0x85 | 3 | Duplicate item ke-6 di stack |
| DUP7 | 0x86 | 3 | Duplicate item ke-7 di stack |
| DUP8 | 0x87 | 3 | Duplicate item ke-8 di stack |
| DUP9 | 0x88 | 3 | Duplicate item ke-9 di stack |
| DUP10 | 0x89 | 3 | Duplicate item ke-10 di stack |
| DUP11 | 0x8A | 3 | Duplicate item ke-11 di stack |
| DUP12 | 0x8B | 3 | Duplicate item ke-12 di stack |
| DUP13 | 0x8C | 3 | Duplicate item ke-13 di stack |
| DUP14 | 0x8D | 3 | Duplicate item ke-14 di stack |
| DUP15 | 0x8E | 3 | Duplicate item ke-15 di stack |
| DUP16 | 0x8F | 3 | Duplicate item ke-16 di stack |

### 7.4 SWAP Operations (Swap Stack Items)
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| SWAP1 | 0x90 | 3 | Swap top dengan item ke-2 |
| SWAP2 | 0x91 | 3 | Swap top dengan item ke-3 |
| SWAP3 | 0x92 | 3 | Swap top dengan item ke-4 |
| SWAP4 | 0x93 | 3 | Swap top dengan item ke-5 |
| SWAP5 | 0x94 | 3 | Swap top dengan item ke-6 |
| SWAP6 | 0x95 | 3 | Swap top dengan item ke-7 |
| SWAP7 | 0x96 | 3 | Swap top dengan item ke-8 |
| SWAP8 | 0x97 | 3 | Swap top dengan item ke-9 |
| SWAP9 | 0x98 | 3 | Swap top dengan item ke-10 |
| SWAP10 | 0x99 | 3 | Swap top dengan item ke-11 |
| SWAP11 | 0x9A | 3 | Swap top dengan item ke-12 |
| SWAP12 | 0x9B | 3 | Swap top dengan item ke-13 |
| SWAP13 | 0x9C | 3 | Swap top dengan item ke-14 |
| SWAP14 | 0x9D | 3 | Swap top dengan item ke-15 |
| SWAP15 | 0x9E | 3 | Swap top dengan item ke-16 |
| SWAP16 | 0x9F | 3 | Swap top dengan item ke-17 |

---

## 💾 8. MEMORY OPERATIONS (Operasi Memory)

Opcodes untuk manipulasi volatile memory selama eksekusi.

| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| MLOAD | 0x51 | 3* | Load word (32 bytes) dari memory |
| MSTORE | 0x52 | 3* | Store word (32 bytes) ke memory |
| MSTORE8 | 0x53 | 3* | Store single byte ke memory |
| MSIZE | 0x59 | 2 | Ukuran aktif memory dalam bytes |
| MCOPY | 0x5E | 3* | Copy memory region (EIP-5656) |

*Gas termasuk potential memory expansion cost

---

## 💽 9. STORAGE OPERATIONS (Operasi Storage)

Opcodes untuk persistent storage di blockchain.

### 9.1 Permanent Storage
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| SLOAD | 0x54 | 100* | Load word dari storage (warm/cold) |
| SSTORE | 0x55 | 100* | Store word ke storage (complex gas) |

### 9.2 Transient Storage (EIP-1153)
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| TLOAD | 0x5C | 100 | Load dari transient storage |
| TSTORE | 0x5D | 100 | Store ke transient storage |

*Transient storage di-reset setiap transaksi selesai

---

## 🔀 10. CONTROL FLOW OPERATIONS (Operasi Control Flow)

Opcodes untuk mengontrol alur eksekusi program.

| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| JUMP | 0x56 | 8 | Unconditional jump ke destination |
| JUMPI | 0x57 | 10 | Conditional jump jika condition ≠ 0 |
| JUMPDEST | 0x5B | 1 | Mark valid jump destination |
| PC | 0x58 | 2 | Program counter sebelum increment |
| GAS | 0x5A | 2 | Gas yang tersisa |
| STOP | 0x00 | 0 | Halt execution (success) |

---

## 📝 11. LOGGING OPERATIONS (Operasi Logging/Events)

Opcodes untuk emit events ke blockchain logs.

| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| LOG0 | 0xA0 | 375* | Log tanpa topics |
| LOG1 | 0xA1 | 750* | Log dengan 1 topic |
| LOG2 | 0xA2 | 1125* | Log dengan 2 topics |
| LOG3 | 0xA3 | 1500* | Log dengan 3 topics |
| LOG4 | 0xA4 | 1875* | Log dengan 4 topics |

*Base cost + 8 gas per byte data + 375 per topic

---

## 📞 12. SYSTEM OPERATIONS / CALLS (Operasi Sistem)

### 12.1 Contract Creation
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| CREATE | 0xF0 | 32000* | Create contract baru |
| CREATE2 | 0xF5 | 32000* | Create dengan deterministic address |

### 12.2 External Calls
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| CALL | 0xF1 | 100* | Call ke external contract |
| CALLCODE | 0xF2 | 100* | Call dengan code contract lain (deprecated) |
| DELEGATECALL | 0xF4 | 100* | Call preserving msg.sender & msg.value |
| STATICCALL | 0xFA | 100* | Read-only call (no state modification) |

### 12.3 Return & Revert
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| RETURN | 0xF3 | 0* | Return data dan halt (success) |
| REVERT | 0xFD | 0* | Revert dengan data (refund gas) |

### 12.4 Self Destruct
| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| SELFDESTRUCT | 0xFF | 5000* | Destroy contract, send ETH (deprecated behavior post-Cancun) |

---

## ❌ 13. INVALID / HALT OPERATIONS

| Opcode | Hex  | Gas | Deskripsi |
|--------|------|-----|-----------|
| INVALID | 0xFE | All | Designated invalid opcode (EIP-141) |

---

## 🚫 14. UNDEFINED / RESERVED OPCODES

Rentang opcode yang tidak didefinisikan dan akan menyebabkan error jika dieksekusi:

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

## 📊 RINGKASAN STATISTIK

| Kategori | Jumlah Opcodes |
|----------|----------------|
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

## 🔧 CATATAN TEKNIS

### Gas Cost Types:
1. **Static Gas** - Biaya tetap (contoh: ADD = 3 gas)
2. **Dynamic Gas** - Biaya bervariasi berdasarkan:
   - Memory expansion
   - Data size
   - Cold/Warm access (EIP-2929)
   - Storage slot status

### Access Lists (EIP-2929):
- **Cold Access**: Pertama kali akses address/slot = lebih mahal
- **Warm Access**: Akses berikutnya = lebih murah

### EIP References:
- EIP-1153: Transient Storage (TLOAD, TSTORE)
- EIP-2929: Gas cost increases for state access
- EIP-4844: Blob transactions (BLOBHASH, BLOBBASEFEE)
- EIP-5656: MCOPY instruction

---

## 📖 REFERENSI

- [evm.codes](https://www.evm.codes) - Interactive reference
- [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
- [EIPs](https://eips.ethereum.org/)
- [go-ethereum source](https://github.com/ethereum/go-ethereum)

---

*Document Version: 1.0*  
*Last Updated: December 2024*  
*Author: Blockchain Engineer Reference*
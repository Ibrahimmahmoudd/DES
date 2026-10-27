# Data Encryption Standard (DES)
## Resources, Rules & Implementation Guide

To ensure everyone is using consistent techniques and following the project requirements, this guide explains the responsibilities, implementation rules, and useful resources for each part of the DES algorithm.

## 1. File I/O & Command Line

**Responsibility:** Read and write input and output files in 8-byte (64-bit) blocks without corrupting binary data.

### Required concepts

- **Command-line arguments (`argc`, `argv`):** Allow the program to receive input filenames, output filenames, and other required parameters.
- **Binary file I/O:** Use `fread()` and `fwrite()` to process binary data.
- **Binary file modes:** Open input files using `"rb"` and output files using `"wb"` to prevent text-mode transformations.

### Resources

- [C Command Line Arguments — TutorialsPoint](https://www.tutorialspoint.com/cprogramming/c_command_line_arguments.htm)
- [Reading and Writing Binary Files in C](https://www.geeksforgeeks.org/c-program-read-write-binary-file/)

### Important considerations

Read each 8-byte block into an `unsigned char` array before converting it into a 64-bit integer. Construct the integer manually using shifts and bitwise OR to ensure that the bytes are interpreted in big-endian order.

---

## 2. Key Schedule & Circular Left Shifts

**Responsibility:** Implement the DES key-schedule mechanism that transforms the original 64-bit key into 16 round keys, each containing 48 bits.

The key schedule prepares the subkeys used by the 16 rounds of DES encryption. It does not encrypt the plaintext directly; instead, it generates the round keys required by the Feistel function.

### 2.1 Key Schedule Overview

The original DES key contains 64 bits, including 8 parity bits. The key schedule applies two permutation operations and rotates two 28-bit halves to generate the 16 round keys.

The process is:

1. Convert the 8 key bytes into a 64-bit integer.
2. Apply **Permuted Choice 1 (PC-1)** to select 56 bits.
3. Split the 56-bit result into two 28-bit halves, `C` and `D`.
4. Circularly rotate both halves according to the DES shift schedule.
5. Combine the rotated halves into a 56-bit value.
6. Apply **Permuted Choice 2 (PC-2)** to select 48 bits.
7. Store the resulting subkey for the current round.
8. Repeat the rotation and subkey-generation steps for all 16 rounds.

### 2.2 PC-1: Permuted Choice 1

PC-1 transforms the original 64-bit key into a 56-bit key by selecting specific bit positions and excluding the eight parity-bit positions.

```c
uint64_t permutedKey = permute_bits(
    originalKey,
    64,
    Permuted_Choice_1,
    56
);
```

**Why is it needed?**

The DES key schedule operates on 56 effective key bits rather than all 64 bits. PC-1 also rearranges the selected bits according to the DES specification.

### 2.3 Splitting the Key into Two Halves

After PC-1, the 56-bit result is divided into two 28-bit values:

```c
uint32_t C = (uint32_t)((permutedKey >> 28) & HALF_MASK);
uint32_t D = (uint32_t)(permutedKey & HALF_MASK);
```

- `C` contains the upper 28 bits.
- `D` contains the lower 28 bits.
- `HALF_MASK` is `0x0FFFFFFF`, which preserves only the lowest 28 bits.

Both halves are rotated independently during every round.

### 2.4 Circular Left Shift

A circular left shift rotates bits to the left while bringing the bits that leave the left side back to the right side.

Unlike an ordinary left shift, a circular shift preserves the 28-bit half's information.

```c
uint32_t circular_left_shift(uint32_t value, int shifts)
{
    value &= HALF_MASK;

    return ((value << shifts) |
            (value >> (DES_HALF_BITS - shifts)))
            & HALF_MASK;
}
```

**Explanation:**

- `value << shifts` moves the bits to the left.
- `value >> (DES_HALF_BITS - shifts)` brings the outgoing bits back from the right side.
- The bitwise OR (`|`) combines the two parts.
- `& HALF_MASK` ensures that the result remains within 28 bits.

The mask is essential because the C data type `uint32_t` has 32 bits, while each DES key half has only 28 bits.

### 2.5 DES Shift Schedule

The number of positions rotated depends on the round:

| Round | Left shifts |
|---|---:|
| 1 | 1 |
| 2–8 | 2 per round |
| 9 | 1 |
| 10–15 | 2 per round |
| 16 | 1 |

The implementation stores these values in `key_shift_sizes` and retrieves the number of shifts for each round.

```c
int shifts = key_shift_sizes[round + 1];

C = circular_left_shift(C, shifts);
D = circular_left_shift(D, shifts);
```

Both halves must use the same shift amount during each round.

### 2.6 PC-2: Permuted Choice 2

After rotating `C` and `D`, they are combined into a 56-bit value:

```c
uint64_t combinedKey = ((uint64_t)C << 28) | D;
```

PC-2 then selects 48 bits from this combined value to generate the current round key.

```c
roundKeys[round] = permute_bits(
    combinedKey,
    56,
    Permuted_Choice_2,
    48
);
```

Each round key is stored in a `uint64_t`, although only its lowest 48 bits are used.

### 2.7 The Bit Permutation Engine

The reusable `permute_bits()` function supports the key schedule by applying the specified permutation table.

```c
uint64_t permute_bits(
    uint64_t input,
    int inputBits,
    const int *table,
    int outputBits
);
```

Parameters:

- `input`: The value whose bits are being rearranged.
- `inputBits`: The number of valid bits in the input.
- `table`: The permutation table specifying which bits to select.
- `outputBits`: The number of bits in the result.

DES permutation tables use **1-based positions counted from the left**. C bitwise shifts count positions from the right, starting at zero. Therefore, the shift amount is calculated as:

```c
int shift = inputBits - table[i];
```

The selected bit is extracted using:

```c
uint64_t bit = (input >> shift) & 1ULL;
```

The output is constructed one bit at a time:

```c
output = (output << 1) | bit;
```

Using `const int *table` allows the function to read different permutation tables without modifying their contents.

### 2.8 Generating All 16 Round Keys

The `generate_subkeys()` function combines the preceding operations to generate the complete set of round keys.

```c
void generate_subkeys(
    const unsigned char key[8],
    uint64_t roundKeys[DES_ROUNDS]
);
```

- `key` contains the original 8 key bytes.
- `roundKeys` receives the 16 generated subkeys.
- Each subkey contains 48 meaningful bits.

The rotations are cumulative: each round begins with the halves produced by the previous round, rather than rotating the original halves independently.
### Resources

- [How to Rotate Bits in C](https://www.geeksforgeeks.org/rotate-bits-of-an-integer/)
- [Bitwise Operators in C](https://www.geeksforgeeks.org/bitwise-operators-in-c-cpp/)
- [Bitwise Hacks](https://www.geeksforgeeks.org/bitwise-hacks-for-competitive-programming/)

---

## 3. Bitwise Permutations Engine

**Responsibility:** Implement the reusable bit-selection and rearrangement logic required by DES.

### Required concepts

- Bit shifting (`<<`, `>>`).
- Bitwise AND (`&`) for extracting individual bits.
- Bitwise OR (`|`) for constructing output values.
- Permutation tables and bit-position indexing.
- Fixed-width integer types.

### Important considerations

DES tables specify positions starting from 1 at the leftmost bit. Convert these positions into C shift amounts before extracting bits.

The same permutation engine can support PC-1, PC-2, the initial permutation, the expansion permutation, and the final permutation by receiving the appropriate table and input/output bit lengths.

### Resources

- [Bitwise Operators in C](https://www.geeksforgeeks.org/bitwise-operators-in-c-cpp/)
- [Bitwise Hacks](https://www.geeksforgeeks.org/bitwise-hacks-for-competitive-programming/)

---

## 4. Feistel Function & S-Boxes

**Responsibility:** Implement the DES round function, which combines expansion, key mixing, substitution, and permutation.

### Required operations

1. Expand the 32-bit right half into 48 bits using the expansion permutation.
2. XOR the expanded result with the 48-bit round key.
3. Divide the result into eight 6-bit chunks.
4. Use the outer bits of each chunk to calculate the S-box row.
5. Use the middle four bits to calculate the S-box column.
6. Look up a 4-bit result from each S-box.
7. Combine the eight 4-bit results into a 32-bit value.
8. Apply the P permutation.

### S-box indexing

For a 6-bit chunk `b5 b4 b3 b2 b1 b0`:

- The row is determined by `b5` and `b0`.
- The column is determined by `b4 b3 b2 b1`.

The eight S-boxes collectively transform 48 bits into 32 bits.

### Resource

- [Bit Masking and Shifting](https://stackoverflow.com/questions/10493411/what-is-bit-masking)

---

## 5. Round Orchestration & Logic

**Responsibility:** Connect the DES components into the complete 16-round Feistel network.

### Required operations

1. Apply the initial permutation to the 64-bit plaintext block.
2. Split the result into two 32-bit halves, `L0` and `R0`.
3. Execute 16 Feistel rounds using the generated round keys.
4. Apply the final permutation to the appropriately ordered halves.
5. Produce the 64-bit ciphertext block.

In a typical DES round:

- The new left half becomes the previous right half.
- The new right half is the previous left half XORed with the Feistel function applied to the previous right half and the current round key.

Be careful with the final-round ordering: standard DES performs the final half swap as part of the final permutation input convention. Implement the round and final-permutation stages consistently with the DES specification.

### Resource

- [Understanding the Feistel Cipher](https://en.wikipedia.org/wiki/Feistel_cipher)

---

## 6. Important Project Rules

### Big-Endian vs. Little-Endian

The DES bit numbering convention must be preserved when converting input bytes to a 64-bit value. Read bytes into an `unsigned char` array and assemble the value explicitly using shifts and bitwise OR.

### Zero External Libraries

Only standard C libraries are permitted, such as:

```c
#include <stdio.h>
#include <stdint.h>
```

No external libraries, threading, or custom header files are allowed.

### One Source File

All submitted implementation code must be combined into one `.c` file.

### No Console Output

The final program must not print anything to the command line. Remove temporary `printf()` debugging and testing statements before submission.

### Binary File Processing

Use `"rb"` for input and `"wb"` for output. Process the input in 8-byte blocks while handling any incomplete final block according to the assignment's specified padding or input-length rules.

# Data Encryption Standard (DES) Implementation

A pure C implementation of the DES algorithm. This project uses only standard C libraries, processes binary files block-by-block, and strictly enforces Big-Endian data alignment.

## Features
* **Zero External Dependencies:** Built entirely with the C Standard Library.
* **Big-Endian Enforcement:** Strict bit-shifting ensures consistent 64-bit block byte order regardless of host architecture.
* **One Source File:** All submitted implementation code must be combined into one `.c` file.
* **Binary File I/O:** Safely reads/writes raw bytes without string-terminator corruption.
* **Automated Decryption:** Automatically reverses the Key Schedule for decryption.

## Compilation
Compile the source code using `gcc` with the `-O3` flag for maximum performance:
```bash
gcc -O3 main.c -o des
```

## Usage
The program requires exactly 4 command-line arguments:
```bash
./des <encrypt|decrypt> <key_file> <input_file> <output_file>
```

## Testing Instructions
Verify the algorithm works flawlessly by generating binary files directly from your terminal and running a full cycle.

**1. Create a binary key and input file**
Use `printf` to write raw hexadecimal bytes to a file:
```bash
printf "\x0f\x15\x71\xc9\x47\xd9\xe8\x59" > key.bin
printf "\x02\x46\x8a\xce\xec\xa8\x64\x20" > input.bin
```

**2. Encrypt the file**
```bash
./des encrypt key.bin input.bin cipher.bin
```
*(Optional: View output using `xxd -p cipher.bin`. It should output `da02ce3a89ecac3b` based on the Stallings test vector).*

**3. Decrypt the file**
```bash
./des decrypt key.bin cipher.bin decrypted.bin
```

**4. Verify the cycle**
Compare the decrypted output against the original input. It should exactly match the input bytes:
```bash
xxd -p decrypted.bin
```
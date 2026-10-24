# Data Encryption Standard

## Resources, Rules & Implementation Guide

To ensure everyone is on the same page and using the most efficient C techniques (specifically bitwise operations), please refer to the resources below based on your assigned task.

### 1. File I/O & Command Line
Your responsible for reading and writing strictly in 8-byte (64-bit) chunks using standard C file operations without corrupting binary data.
* **Command Line Arguments (`argc`, `argv`):** [C Command Line Arguments (TutorialsPoint)](https://www.tutorialspoint.com/cprogramming/c_command_line_arguments.htm)
* **Binary File I/O:** [Reading and Writing Binary Files in C (`fread`, `fwrite`)](https://www.geeksforgeeks.org/c-program-read-write-binary-file/)
* **Tip:** Always open files in binary mode using `"rb"` for reading and `"wb"` for writing to prevent the OS from modifying line-ending bytes.

### 2. Key Schedule & Circular Shifts
Your part is implementing a 28-bit Circular Left Shift. Standard C `<<` shifts bits into oblivion; you must catch the outgoing bits and bring them back to the right side.
* **Bitwise Rotation/Circular Shift:** [How to Rotate Bits in C](https://www.geeksforgeeks.org/rotate-bits-of-an-integer/)
* **Tip:** Since you are rotating a 28-bit number (not a standard 32-bit integer), your rotation logic will look something like this:
  `rotated = ((half << shift_amount) | (half >> (28 - shift_amount))) & 0x0FFFFFFF;`

### 3. Bitwise Permutations Engine
You are building the core engine that replaces the traditional "swap" function. You will use bit masks to isolate bits and shifts to move them.
* **Mastering Bitwise Operations:** [Bitwise Operators in C (GeeksforGeeks)](https://www.geeksforgeeks.org/bitwise-operators-in-c-cpp/)
* **Setting, Clearing, and Toggling Bits:** [Bitwise Hacks](https://www.geeksforgeeks.org/bitwise-hacks-for-competitive-programming/)
* **Tip:** Remember that DES tables are 1-indexed. When pulling a bit position from a table, always subtract 1 to align with C's 0-indexed logic.

### 4. Feistel Function & S-Boxes
You need to slice a 48-bit number into eight 6-bit chunks and extract specific bits for row and column indexing.
* **Extracting Bits (Masking):** [How to Mask and Shift Bits](https://stackoverflow.com/questions/10493411/what-is-bit-masking)
* **Tip:** When dealing with the 6-bit chunks, use `& 0x20` to check the leftmost bit and `& 0x01` to check the rightmost bit for your S-Box row calculations.

### 5. Round Orchestration & Logic
Your job is to tie everything together. You need a deep understanding of the overall Feistel Network architecture.
* **DES Architecture Flowchart:** [Understanding the Feistel Cipher (Wikipedia)](https://en.wikipedia.org/wiki/Feistel_cipher)
* **Tip:** Be extremely careful with the final round (Round 16). In a standard Feistel network, the Left and Right halves are swapped after every round, **except** after the 16th round.

### Important to Keep in mind
Big-Endian vs Little-Endian: DES strictly assumes data is Big-Endian, but our Intel/AMD computers are Little-Endian. Person 1 (File I/O), you CANNOT just fread() directly into a 64-bit integer, or the computer will reverse the bytes! You must read the 8 bytes into a standard char array first, and then manually bit-shift them into the 64-bit variable to force the Big-Endian order. The rest of us are safe because C bitwise shifts handle the values logically regardless of the system.

Zero External Libraries & One File: We are only allowed to use standard C libraries like <stdio.h> and <stdint.h>. No external libraries, no threads, and absolutely NO custom header files. Everything must be merged into one single .c file for submission.

NO Console Output: Do not leave any printf() statements in your code. The project rules strictly state we are not allowed to print any output to the command line when the program runs.

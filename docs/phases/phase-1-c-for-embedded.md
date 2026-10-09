# Phase 1: C for Embedded Systems

**Time:** 1 week if you know C or C++. 2 weeks if you know Rust, Go, or a similar language. 4–6 weeks if you know only managed or scripting languages. Do this phase in parallel with phase 0.
**Prerequisites:** Programming experience in at least one language. Knowledge of C is not necessary.
**Hardware:** None. Do all exercises on your PC with a host compiler and with `arm-none-eabi-gcc`.

## Goal

At the end of this phase, you can:

- Write correct C23 code, and know which C23 features older code and older compilers do not have.
- Use `volatile`, fixed-width integer types, and bit operations correctly.
- Access a hardware register through a pointer to a fixed address.
- Describe a register block with a `struct` and make sure that the layout is correct.
- Explain where the compiler puts code and data (`.text`, `.rodata`, `.data`, `.bss`).
- Find integer promotion and undefined behavior bugs.

## Reading

If you do not know C, start with topic 1.0 and its reading.

| Source | Part |
|---|---|
| Oualline, *Bare Metal C* | The chapters about C basics, bit operations, and memory |
| Gustedt, *Modern C*, 3rd edition | Use as a reference for C23 rules (integer types, conversions, `restrict`, new C23 features) |
| Barr, *Embedded C Coding Standard* | Read all. It is short. |
| Quantum Leaps (Samek), lessons 1–10 | Lessons about pointers, bit operations, `volatile`, and functions at the assembly level |

## Topics

### 1.0 Learn the C basics (only if you do not know C)

Skip this topic if you can already write C programs with pointers, arrays, strings, and structs.

**Reading (select one main book):**

| Source | Comment |
|---|---|
| K. N. King, *C Programming: A Modern Approach*, 2nd edition | Complete and clear. The best main book for a new C programmer. It uses C99: learn the C23 changes in topic 1.13. |
| Beej's Guide to C Programming (free, beej.us) | Short and practical |
| Gustedt, *Modern C*, 3rd edition (free PDF) | Level 1 teaches the basics with modern C, including C23 |
| Kernighan and Ritchie, *The C Programming Language*, 2nd edition | Short, classic, but old (C89). Use it as a second book. |

**Learn these subjects, in this order:**

1. The compilation model: preprocessor, compiler, linker. Header files (`.h`) and source files (`.c`).
2. Types, operators, control flow, and functions.
3. Pointers: the address operator `&`, the dereference operator `*`, pointer arithmetic, `NULL`.
4. Arrays. An array name converts to a pointer to its first element. C does not check array bounds.
5. Strings: a string is a `char` array that ends with `'\0'`. `strlen`, `strcpy`, `snprintf`, and the dangers of buffer overflow.
6. Structs, unions, enums, and `typedef`.
7. The stack and the heap. `malloc` and `free` on the PC. Memory leaks and dangling pointers.
8. Undefined behavior: the program can do anything, and the compiler does not warn you.

**Practice:** write these programs on your PC:

1. A function that reverses a string in place.
2. A dynamic array of integers that grows with `realloc`.
3. A singly linked list with insert, remove, and free functions.
4. A program that reads a text file and counts the words.

Compile with `-Wall -Wextra -fsanitize=address,undefined`. The sanitizers find many memory errors.

### 1.1 C compared with other languages

| Feature in other languages | Examples | C replacement |
|---|---|---|
| Classes, methods | C#, Java, C++, Python | `struct` and functions that take a pointer to the `struct` |
| Namespaces, packages, modules | C#, Java, C++, Rust, Python | Name prefixes: `uart_init()`, `uart_write()` |
| Overloading | C#, Java, C++ | Different function names |
| References | C#, Java, C++, Rust | Pointers |
| Garbage collector, RAII, destructors | C#, Java, Python, Go, C++, Rust | Explicit cleanup. Use `goto cleanup;` for error paths. |
| Exceptions, `Result<T, E>`, multiple return values | C#, Java, C++, Python, Rust, Go | Return an error code. Return values through pointer arguments. |
| Generics, templates | C#, Java, C++, Rust, Go | Macros, `void *`, or code generation. Use them carefully. |
| Interfaces, virtual functions, traits | C#, Java, Go, C++, Rust | A `struct` of function pointers (an "ops" table) |
| Lists, vectors, dynamic arrays | All | Fixed-size arrays and static memory pools |
| `private` fields | C#, Java, C++, Rust | Opaque types: declare `struct uart;` in the header, define it in the `.c` file |
| Lambdas, closures | C#, Java, Python, C++, Rust, Go | A function pointer and a `void *context` argument (topic 1.12) |
| String type | All | A `char` array that ends with `'\0'`. You manage the buffer size. |
| Bounds checks, overflow checks | C#, Java, Python, Rust (debug builds) | None. You must make the checks yourself. |

**Notes for programmers of managed languages (C#, Java, Python, JavaScript):**

- No runtime protects you. An error in pointer use can corrupt memory far from the error and cause a failure much later.
- A `struct` assignment copies all of its bytes. An array is not copied when you pass it to a function: the function gets a pointer.
- Integers have fixed sizes and can wrap around. Signed overflow is undefined behavior (topic 1.8).

**Notes for C++ programmers:**

- `void *` converts to any object pointer without a cast in C.
- A `struct` tag is in its own namespace. Use `typedef` if you want to write the type without `struct`.
- `const` at file scope has external linkage in C (internal linkage in C++).
- In C23, an empty parameter list `f()` means "no parameters", as in C++. Before C23, it means "unknown parameters". Much existing code writes `f(void)`. Both forms are correct in C23.

**Notes for Rust programmers:**

- The compiler does not check ownership, lifetimes, or data races. You must follow these rules yourself, and document them in comments.
- `unsafe` code in Rust is the normal state of all C code.

### 1.2 Fixed-width integer types

- Include `<stdint.h>`. In C23, `bool`, `true`, and `false` are keywords. Before C23, include `<stdbool.h>`. Older code and vendor headers often include it.
- Use `uint8_t`, `uint16_t`, `uint32_t`, `int32_t` for data that has a fixed size, for example registers and protocol fields.
- On Cortex-M, `int` is 32 bits. On an 8-bit or 16-bit MCU, `int` can be 16 bits. Do not use `int` for values that depend on the size.
- Use `size_t` for sizes and array indexes.

### 1.3 Bit operations

| Operation | Code |
|---|---|
| Set bit `n` | `reg \|= (1U << n);` |
| Clear bit `n` | `reg &= ~(1U << n);` |
| Toggle bit `n` | `reg ^= (1U << n);` |
| Test bit `n` | `if (reg & (1U << n))` |
| Write a field of width `w` at position `p` | `reg = (reg & ~(mask << p)) \| ((value & mask) << p);` with `mask = (1U << w) - 1U` |

- Always use unsigned constants (`1U`) in shifts. `1 << 31` is undefined behavior because `1` is a signed `int`.
- A read-modify-write (`reg |= x`) is three operations: read, change, write. An interrupt can occur between them. Phase 3 explains the problem and the solutions.

### 1.4 `volatile`

- `volatile` tells the compiler that every read and every write is important. The compiler must not remove, combine, or reorder `volatile` accesses.
- Use `volatile` for:
  - Hardware registers
  - Variables that an interrupt handler changes and the main code reads
- `volatile` does **not** make an operation atomic. It does **not** add a memory barrier for other CPUs or for DMA.
- `const volatile` describes a read-only register: the program must not write it, but its value can change.

### 1.5 Memory-mapped registers

A peripheral register is a fixed address in memory. You read and write it like memory.

```c
#define GPIOA_ODR (*(volatile uint32_t *)0x40020014U)

GPIOA_ODR |= (1U << 5);  /* Set pin PA5 high. */
```

A better method describes the whole register block with a `struct`:

```c
typedef struct {
    volatile uint32_t MODER;    /* 0x00 */
    volatile uint32_t OTYPER;   /* 0x04 */
    volatile uint32_t OSPEEDR;  /* 0x08 */
    volatile uint32_t PUPDR;    /* 0x0C */
    volatile uint32_t IDR;      /* 0x10 */
    volatile uint32_t ODR;      /* 0x14 */
    volatile uint32_t BSRR;     /* 0x18 */
    volatile uint32_t LCKR;     /* 0x1C */
    volatile uint32_t AFR[2];   /* 0x20, 0x24 */
} gpio_regs_t;

#define GPIOA ((gpio_regs_t *)0x40020000U)
```

The CMSIS device header for the STM32F411 contains these definitions for all peripherals. In phase 2 you write some of them yourself, then you change to the CMSIS header.

- Use `static_assert(offsetof(gpio_regs_t, BSRR) == 0x18);` to do a check of the layout. In C23, `static_assert` is a keyword and the message is optional. Before C23, write `_Static_assert(expression, "message");`.
- **Do not use bit-fields for registers.** The C standard does not define the bit order or the access width of bit-fields.

### 1.6 Memory layout and sections

| Section | Content | Location |
|---|---|---|
| `.text` | Code | Flash |
| `.rodata` | `const` data, string literals | Flash |
| `.data` | Initialized global and `static` variables | Stored in flash, copied to SRAM at startup |
| `.bss` | Global and `static` variables with no initializer or with zero | SRAM, set to zero at startup |
| Stack | Local variables, return addresses | SRAM |
| Heap | `malloc` | SRAM. Usually not used in this course. |

- A `const` array goes to flash. A non-`const` array that has an initializer uses flash **and** SRAM.
- The startup code copies `.data` and clears `.bss`. You write this code in phase 2.

### 1.7 No dynamic memory

- `malloc` on a small MCU can fail because of fragmentation, at a time that you cannot predict.
- Use static allocation: global arrays, `static` local arrays, and fixed-size pools.
- If an object has a fixed number of instances, allocate all instances at compile time.

### 1.8 Integer promotion and undefined behavior

- Arithmetic on `uint8_t` and `uint16_t` values first converts them to `int`. Example:

  ```c
  uint8_t a = 0x0F;
  if ((uint8_t)~a == 0xF0) { }  /* Correct. */
  if (~a == 0xF0) { }           /* False: ~a is the int 0xFFFFFFF0. */
  ```

- Signed overflow is undefined behavior. Use unsigned types for counters that wrap around.
- Shifting by a value equal to or greater than the type width is undefined behavior.
- Reading memory through a pointer of a different type breaks the strict aliasing rule. Use `memcpy` to convert bytes to a value. The compiler optimizes a small `memcpy` to one load.
- Enable the warnings: `-Wall -Wextra -Wconversion -Wshadow`. Enable `-fsanitize=undefined` for host tests.

### 1.9 Endianness and byte order

- Cortex-M is little-endian: the least significant byte is at the lowest address.
- Many sensors and network protocols send the most significant byte first (big-endian).
- Build values from bytes with shifts. This code works on all CPUs:

  ```c
  uint16_t value = (uint16_t)((buf[0] << 8) | buf[1]);
  ```

### 1.10 Fixed-point arithmetic

- Small MCUs often have no floating-point unit (FPU). The STM32F411 has a single-precision FPU, but many sensor drivers use integers.
- A fixed-point number stores a fraction as an integer with an implied scale. Example: temperature in units of 0.01 °C. `2345` means 23.45 °C.
- The BME280 datasheet gives integer compensation formulas. You use them in phase 4.

### 1.11 Modules and linkage

- One module = one `.h` file (interface) and one `.c` file (implementation).
- Use `static` for functions and variables that the module does not export.
- Use `extern` declarations only in headers.
- Use `static inline` for small functions in headers.
- Use include guards or `#pragma once`.

### 1.12 Function pointers and callbacks

```c
typedef void (*uart_rx_callback_t)(uint8_t byte, void *context);
```

- Pass a `void *context` with every callback. This replaces the captured state of a lambda or a closure.
- An "ops" table replaces an interface:

  ```c
  typedef struct {
      int (*write)(void *ctx, uint8_t addr, const uint8_t *data, size_t len);
      int (*read)(void *ctx, uint8_t addr, uint8_t *data, size_t len);
      void *ctx;
  } i2c_bus_t;
  ```

  In phase 4, the BME280 driver uses this table. You can then test the driver on your PC with a fake bus.

### 1.13 C23 features for embedded code

The course uses C23 (ISO/IEC 9899:2024), the latest published C standard. Compile with `-std=c23`. You need GCC 14 or newer, or clang 18 or newer. These features are useful in embedded code:

| Feature | Example | Use |
|---|---|---|
| `nullptr` | `uart_t *u = nullptr;` | A null pointer constant with its own type. Use it instead of `NULL`. |
| `constexpr` objects | `constexpr uint32_t BAUD = 115200;` | Typed compile-time constants. Use them instead of many `#define` constants. |
| Binary literals and digit separators | `0b0000'0011'0000'0000` | Readable register masks |
| Enums with a fixed type | `enum gpio_mode : uint8_t { GPIO_IN, GPIO_OUT };` | Exact size for enum values in structs and protocols |
| Attributes | `[[nodiscard]] int i2c_read(...);` | `[[nodiscard]]` makes the compiler warn when a caller ignores an error code. Also `[[maybe_unused]]` and `[[fallthrough]]`. |
| `typeof` | `#define SWAP(a, b) do { typeof(a) t = (a); (a) = (b); (b) = t; } while (0)` | Type-safe macros |
| Empty initializer | `uint8_t buf[16] = {};` | Set all elements to zero |
| `bool`, `static_assert` as keywords | `static_assert(sizeof(frame_t) == 8);` | No extra headers. The message is optional. |
| `#embed` | `static const uint8_t font[] = { #embed "font.bin" };` | Put a binary file (font, image) in flash. Needs GCC 15 or newer, or clang 19 or newer. |

**Compatibility:**

- Vendor headers and libraries (CMSIS, HAL drivers, FreeRTOS, FatFs) use C99 or C11. They normally compile in C23 mode. If a vendor file does not compile, compile only that file with an older standard, for example `-std=c17`.
- If a vendor framework sets its own C standard (for example ESP-IDF), follow the framework.
- Much existing code and many examples on the internet use C99 or C11. You must be able to read them. The notes in topics 1.1, 1.2, and 1.5 show the differences.

## Exercises

Do these exercises on your PC. Use a host GCC (version 14 or newer) or clang (version 18 or newer). Compile with `-std=c23 -Wall -Wextra -Wconversion -fsanitize=undefined`.

### Exercise 1.1: Bit operations

Write and test these functions:

```c
uint32_t bit_set(uint32_t reg, unsigned bit);
uint32_t bit_clear(uint32_t reg, unsigned bit);
uint32_t field_write(uint32_t reg, unsigned pos, unsigned width, uint32_t value);
uint32_t field_read(uint32_t reg, unsigned pos, unsigned width);
```

Test edge cases: bit 0, bit 31, width 32.

### Exercise 1.2: `volatile` and the optimizer

1. Open Compiler Explorer (godbolt.org). Select "ARM GCC (none)" and the flags `-O2 -mcpu=cortex-m4 -mthumb`.
2. Write a loop that waits until a global variable `flag` is not zero.
3. Compare the assembly with and without `volatile`. Explain the difference.
4. Write a function that writes a register two times. Compare the assembly with and without `volatile`.

### Exercise 1.3: Register struct

1. Write the `gpio_regs_t` struct from topic 1.5.
2. Add `static_assert` checks for the offsets of all fields.
3. Allocate a `gpio_regs_t` variable on your PC. Write functions `gpio_set_mode(gpio_regs_t *port, unsigned pin, unsigned mode)` and `gpio_write(gpio_regs_t *port, unsigned pin, bool level)`. Test them against the fake port.

### Exercise 1.4: Sections

1. Write a file with these variables: a `const` array, an initialized global array, a global array with no initializer, a `static` local variable, and a string literal.
2. Compile it with `arm-none-eabi-gcc -c -mcpu=cortex-m4 -mthumb -O2`.
3. Run `arm-none-eabi-objdump -h` and `arm-none-eabi-nm -S` on the object file. Find each variable and its section.
4. Run `arm-none-eabi-size`. Explain the `text`, `data`, and `bss` columns.

### Exercise 1.5: Integer promotion

Predict the result of each expression. Then test it.

```c
uint8_t a = 200, b = 100;
uint8_t c = a + b;
int d = a + b;
uint16_t e = 0xFFFF;
uint32_t f = e * e;   /* Is this defined behavior? */
int8_t g = -1;
unsigned h = 1;
bool i = g < h;
```

### Exercise 1.6: Ring buffer

Write a ring buffer module. You use it in phase 4 for the UART driver.

- Fixed capacity (a power of 2), static storage, no `malloc`.
- Functions: `rb_init`, `rb_put`, `rb_get`, `rb_count`, `rb_is_empty`, `rb_is_full`.
- One writer and one reader. The writer is an interrupt handler and the reader is the main loop (or the reverse). The writer changes only the head index. The reader changes only the tail index.
- Write unit tests: empty, full, wrap-around, and the index overflow of a `uint32_t` counter.

### Exercise 1.7: Byte order

Write `uint16_t be16_read(const uint8_t *p)`, `uint32_t le32_read(const uint8_t *p)`, and the matching write functions. Test them.

### Exercise 1.8: Opaque driver interface

1. Define an `i2c_bus_t` ops table like the one in topic 1.12.
2. Write a fake bus that stores register values in an array.
3. Write a small "device driver" that reads a chip ID from register `0xD0` through the bus. Test it with the fake bus.

## Done criteria

- [ ] If I started with topic 1.0: my four practice programs run without errors from the sanitizers.
- [ ] I can set, clear, toggle, and read bits and fields without undefined behavior.
- [ ] I can explain what `volatile` does and what it does not do.
- [ ] I can describe a register block with a `struct` and do a check of its layout at compile time.
- [ ] I can find the section of a variable with `objdump` or `nm`.
- [ ] I can predict the result of integer promotion.
- [ ] My ring buffer passes all its tests.
- [ ] I can write a driver against an ops table and test it with a fake.

## Next

[Phase 2: How a microcontroller starts](phase-2-boot.md)

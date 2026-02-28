# 🖥️ From Nand to Tetris — Building a Modern Computer From First Principles

> *"The most beautiful thing we can experience is the mysterious. It is the source of all true art and science."* — Albert Einstein

This repository contains my complete implementation of all 12 projects from the [Nand to Tetris](https://www.nand2tetris.org/) course (Parts I & II), taught by Noam Nisan and Shimon Schocken. Starting from a single primitive logic gate — **NAND** — I built a fully functioning computer, complete with an assembler, virtual machine, compiler, operating system, and a playable game.

**Every abstraction layer. Every line. From scratch.**

---

## 🗺️ The Journey at a Glance

```
NAND Gate
   │
   ▼
Boolean Logic Gates  ──► Arithmetic Unit (ALU)  ──► Memory (RAM)
                                                         │
                                                         ▼
                                              Full Computer (CPU + ROM + RAM)
                                                         │
                                                         ▼
                                                    Assembler
                                                         │
                                                         ▼
                                                  VM Translator
                                                         │
                                                         ▼
                                            Jack Compiler (Syntax + Code Gen)
                                                         │
                                                         ▼
                                              Jack OS (8 modules)
                                                         │
                                                         ▼
                                            Tic-Tac-Toe Game in Jack 🎮
```

---

## 📂 Repository Structure

```
nand2tetris/
├── projects/
│   ├── 01/   Boolean Logic
│   ├── 02/   Boolean Arithmetic
│   ├── 03/   Sequential Logic & Memory
│   ├── 04/   Machine Language
│   ├── 05/   Computer Architecture
│   ├── 06/   Assembler
│   ├── 07/   VM I — Stack Arithmetic
│   ├── 08/   VM II — Program Control
│   ├── 09/   High-Level Language (Tic-Tac-Toe)
│   ├── 10/   Compiler I — Syntax Analysis
│   ├── 11/   Compiler II — Code Generation
│   └── 12/   Operating System
└── README.md
```

---

## 🔩 Part I — Hardware (Projects 1–5)

### Project 1 — Boolean Logic
**What I built:** 15 elementary logic gates using only NAND gates as primitives.

`NOT` `AND` `OR` `XOR` `MUX` `DMUX` `MUX4Way` `MUX8Way` `DMux4Way` `DMux8Way` and their 16-bit variants.

**Key insight:** Every logic gate in existence can be constructed from NAND gates alone. NAND is *functionally complete* — it can simulate any Boolean function. This single primitive is the atom from which the entire digital universe is built.

**What this means in practice:**
```
NOT(x)     = NAND(x, x)
AND(x, y)  = NOT(NAND(x, y))
OR(x, y)   = NAND(NOT(x), NOT(y))
```
The `if` statements, comparisons, and logical operations in every C#, Python, or Java program you've ever written ultimately execute as cascades of NAND gates switching billions of times per second.

---

### Project 2 — Boolean Arithmetic
**What I built:** Half Adder → Full Adder → 16-bit Adder → Incrementer → ALU

The **Arithmetic Logic Unit (ALU)** is the mathematical heart of the CPU. It takes two 16-bit inputs and a set of 6 control bits, and produces any of the following outputs:

```
0, 1, -1, x, y, !x, !y, -x, -y
x+1, y+1, x-1, y-1, x+y, x-y, y-x
x&y, x|y
```

**Key insight:** Subtraction is just addition with a negative number. Negative numbers are represented using **two's complement**: flip all bits and add 1. This is why integer overflow wraps around, why `-1` in binary is `1111111111111111`, and why the range of a signed 16-bit integer is -32768 to 32767.

**What this means in practice:** Every arithmetic operation in your programs — including floating point via IEEE 754 — ultimately reduces to these ALU operations. Understanding this explains why integer overflow bugs exist and why certain numeric types have the ranges they do.

---

### Project 3 — Sequential Logic & Memory
**What I built:** 1-bit register → 16-bit register → RAM8 → RAM64 → RAM512 → RAM4K → RAM16K → Program Counter

The fundamental breakthrough here is the **D Flip-Flop** — a circuit that can *remember* a single bit. By chaining flip-flops, you get registers; by arranging registers with address logic, you get RAM.

**Key insight:** All memory in a computer — registers, cache, RAM — is built from flip-flops. The difference between them is just speed and size:

| Memory Type | Access Time | Size | Built From |
|-------------|-------------|------|-----------|
| CPU Registers | ~0.3ns | ~KB | Flip-flops |
| L1 Cache | ~1ns | ~64KB | SRAM (flip-flops) |
| RAM | ~100ns | ~GBs | DRAM (capacitors) |
| SSD | ~100μs | ~TBs | Flash cells |

**What this means in practice:** Local variables in C# are stored in registers or stack (fast). Heap-allocated objects live in RAM (slower). The GC compacts heap memory to reduce fragmentation — this is why GC pauses exist.

---

### Project 4 — Machine Language
**What I built:** Two programs written in raw Hack assembly language.

- `Mult.asm` — multiplies two numbers using repeated addition
- `Fill.asm` — fills/clears the screen based on keyboard input (memory-mapped I/O)

The Hack CPU has just two instruction types:

```
@value          // A-instruction: load value into A register
dest=comp;jump  // C-instruction: compute something, optionally store and jump
```

**Key insight:** Memory-mapped I/O — the screen (address 16384–24575) and keyboard (address 24576) are just memory locations. Writing to screen memory changes pixels. Reading from keyboard memory tells you which key is pressed. There is no magic — just memory.

**What this means in practice:** Every high-level construct you use — loops, conditionals, function calls, objects — gets compiled down to sequences of instructions exactly like these. When you look at a disassembler output or a CPU profiler, this is what you're seeing.

---

### Project 5 — Computer Architecture
**What I built:** CPU → Memory chip → Full Hack Computer

The complete computer consists of:
- **CPU** — fetches, decodes, and executes instructions
- **Instruction Memory (ROM)** — stores the program
- **Data Memory (RAM)** — stores data + screen + keyboard

The **fetch-decode-execute cycle** runs billions of times per second:
```
1. FETCH    → Read instruction at address stored in Program Counter
2. DECODE   → Figure out what type of instruction it is
3. EXECUTE  → Perform the operation, update registers/memory
4. REPEAT   → Increment PC, go to step 1
```

**Key insight:** The CPU doesn't know about objects, classes, or garbage collection. It only knows: load from memory, store to memory, compute, jump. Everything else is an abstraction built on top of this.

**What this means in practice:** CPU-bound vs I/O-bound performance. Branch misprediction penalties. Why cache locality matters. Why virtual function calls are slower than direct calls. All of these trace back to this level.

---

## 💾 Part II — Software (Projects 6–12)

### Project 6 — Assembler
**What I built:** A complete assembler in Python that translates Hack assembly (`.asm`) into binary machine code (`.hack`).

**Features:**
- Two-pass parser (first pass builds symbol table, second pass translates)
- Symbol resolution for labels and variables
- Handles all A-instructions and C-instructions
- Supports the full Hack assembly specification

**Key insight:** The assembler performs two passes because of **forward references** — a `GOTO LOOP` instruction might appear before the label `(LOOP)` is defined. The first pass builds a complete symbol table; the second pass translates. This is exactly how C# handles forward declarations — you can call a method defined later in the same file.

```
Assembly          →    Binary
@2                →    0000000000000010
D=A               →    1110110000010000
@3                →    0000000000000011
D=D+A             →    1110000010010000
@0                →    0000000000000000
M=D               →    1110001100001000
```

**What this means in practice:** The .NET JIT compiler does something equivalent — it takes IL bytecode and produces native machine instructions. Understanding this two-pass approach explains how linkers resolve symbols across multiple compiled files.

---

### Projects 7 & 8 — VM Translator
**What I built:** A VM translator that converts stack-based VM code into Hack assembly.

This is the most important project for understanding how modern runtimes like the CLR and JVM work.

**The Stack Machine:**
```
// VM code for: x = a + b * c
push a
push b
push c
call Math.multiply 2    // pops b and c, pushes result
add                     // pops a and (b*c), pushes sum
pop x
```

The VM has 8 memory segments:

| Segment | Purpose | C# Equivalent |
|---------|---------|---------------|
| `local` | Local variables | Local vars in a method |
| `argument` | Function arguments | Method parameters |
| `this` | Current object fields | `this.field` |
| `that` | Array elements | `array[i]` |
| `static` | Class-level variables | `static` fields |
| `constant` | Integer literals | `42`, `true`, `null` |
| `temp` | Temporary storage | Compiler temporaries |
| `pointer` | Base addresses | Pointers |

**The Call Stack:**
When a function is called, a **stack frame** is pushed:
```
┌─────────────────┐
│   return addr   │  ← where to go after return
│   saved LCL     │  ← caller's local segment
│   saved ARG     │  ← caller's argument segment
│   saved THIS    │  ← caller's this pointer
│   saved THAT    │  ← caller's that pointer
│   local vars    │  ← callee's local variables
│   working stack │  ← expression evaluation
└─────────────────┘
```

**Key insight:** This is exactly what the .NET CLR does for every method call. The "call stack" you see in a debugger or exception trace is a list of all active stack frames. A `StackOverflowException` happens when you recurse so deeply that you run out of stack memory.

**What this means in practice:**
- Why recursion can be expensive (each call allocates a new stack frame)
- Why tail-call optimization matters (reuse the current frame instead of creating a new one)
- What the debugger call stack is showing you
- Why local variables "disappear" after a method returns

---

### Projects 10 & 11 — Jack Compiler
**What I built:** A complete two-stage compiler for the Jack language.

**Stage 1 — Syntax Analyzer (Project 10):**
```
Jack source code  →  Tokenizer  →  Token stream  →  Parser  →  Parse tree (XML)
```

**Stage 2 — Code Generator (Project 11):**
```
Parse tree  →  Symbol Table  →  VM Code
```

**The Compilation Pipeline:**

```
class Square {
    field int x, y;           // → symbol table: x=field[0], y=field[1]
    
    method void draw() {       // → function Square.draw 0
        do Screen.drawRect(    // → push pointer 0 (this)
            x, y, x+10, y+10  // → push this 0, push this 1, etc.
        );                     // → call Screen.drawRectangle 4
    }
}
```

**The Symbol Table:**
Every variable is tracked with 4 attributes:

| Name | Type | Kind | Index |
|------|------|------|-------|
| `x` | `int` | `field` | `0` |
| `y` | `int` | `field` | `1` |
| `size` | `int` | `field` | `2` |
| `this` | `Square` | `argument` | `0` |

**Key insight:** When the compiler sees `x`, it looks up the symbol table, finds `field[0]`, and generates `push this 0`. There's no magic — variable names only exist at compile time. By runtime, everything is just memory addresses and offsets.

**Object construction translates to:**
```jack
let s = Square.new(10, 20, 30);
```
```vm
push constant 3          // allocate 3 fields
call Memory.alloc 1      // get heap memory
pop pointer 0            // set THIS to new object
push constant 10         // push args
push constant 20
push constant 30
call Square.new 3        // call constructor
```

**What this means in practice:**
- Why `new` has a performance cost (heap allocation + constructor call)
- Why object fields have a fixed offset (they're array indices into allocated memory)
- How scope works (inner scope creates a new symbol table layer)
- Why the C# compiler can detect unused variables and type mismatches

---

### Project 12 — Operating System
**What I built:** A complete OS in 8 Jack modules.

#### `Memory.jack` — Heap Allocator
Implements a **free list** memory allocator:
```
Heap: [block1|block2|free|block3|free|free|block4]
                     ↑                 ↑
                  freeList          next free
```
- `alloc(size)` — finds a free block big enough, splits it, returns a pointer
- `deAlloc(ptr)` — adds the block back to the free list

**What this means in practice:** This is a simplified version of what .NET's GC manages. The GC adds garbage collection (automatic dealloc), generational collection (most objects die young), and compaction (defragmentation). Object pooling is a performance optimization that bypasses alloc/deAlloc entirely.

#### `Math.jack` — Arithmetic
Implements multiplication using **bit shifting** (no hardware multiply instruction):
```
x * y = sum of x * 2^i for each bit i set in y
```
```jack
while (i < 16) {
    if (bit i of y is set) { sum += shiftedX; }
    shiftedX = shiftedX + shiftedX;  // multiply by 2
    i++;
}
```
**What this means:** Powers of 2 are special in computing because multiplying/dividing by them is a single bit shift operation — the fastest possible arithmetic.

#### `String.jack` — Mutable String Buffer
Implements strings as a char array with a length counter.

**What this means in practice:** This is exactly why `System.String` in C# is immutable and why `StringBuilder` exists. Concatenating strings in a loop:
```csharp
string s = "";
for (int i = 0; i < 1000; i++) s += i;  // O(n²) — creates 1000 new strings!
```
vs:
```csharp
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++) sb.Append(i);  // O(n) — one buffer
```

#### `Screen.jack` — Graphics
Implements **Bresenham's line algorithm** — draws lines using only integer arithmetic (no floating point):
```
while not at endpoint:
    plot current pixel
    if error > 0: step in major axis, adjust error
    else: step in minor axis, adjust error
```

#### `Output.jack` — Text Rendering
Each character is stored as an 11-row bitmap. Rendering a character means copying its bitmap into screen memory at the cursor position.

#### `Keyboard.jack` — Input
Reads from memory address 24576 — the memory-mapped keyboard register. Non-blocking `keyPressed()` and blocking `readChar()`.

#### `Sys.jack` — Bootstrap
The entry point that initializes all OS modules in order and calls `Main.main()`.

---

### Project 9 — Tic-Tac-Toe Game 🎮
**What I built:** A fully playable two-player Tic-Tac-Toe game in Jack.

**Features:**
- Graphical 3×3 board rendered with the Screen API
- Arrow key navigation with a selection marker
- X and O rendered graphically (lines and circles)
- Win detection across all 8 possible lines
- Animated win line drawn through winning cells
- Draw detection
- Winner announcement

**Controls:**
```
← → ↑ ↓   Navigate the board
  Enter    Place your mark
```

---

## 🧠 What This Course Teaches That Most Developers Never Learn

### The Full Stack of Abstraction

| Layer | Technology | Built In |
|-------|-----------|----------|
| Game / App | Tic-Tac-Toe in Jack | Project 9 |
| OS | Memory, I/O, Math | Project 12 |
| Compiler | Jack → VM | Projects 10-11 |
| VM | Stack machine | Projects 7-8 |
| Assembler | Assembly → Binary | Project 6 |
| Machine Language | Hack assembly | Project 4 |
| Computer | CPU + RAM | Project 5 |
| Memory | RAM, registers | Project 3 |
| Arithmetic | ALU | Project 2 |
| Logic | Gates | Project 1 |
| Primitive | NAND gate | Given |

### Key Realizations

**1. Variables don't exist at runtime.**
Variable names are compile-time constructs. By the time your code runs, `x` is just `memory[baseAddress + 2]`. The compiler's symbol table is what makes names meaningful.

**2. Objects are just memory blocks with a pointer.**
`new MyClass()` allocates a chunk of heap memory and returns its address. Fields are accessed by fixed offsets from that address. `this` is just argument 0 passed to every method.

**3. The call stack is a physical data structure.**
Every method call pushes a frame. Every return pops one. Stack overflow is literally running out of stack memory. The debugger call stack is showing you real memory.

**4. The CPU knows nothing about your abstractions.**
No objects, no classes, no GC, no types. Just: load, store, add, subtract, compare, jump. Everything else is built on top of these 7 operations.

**5. Performance intuitions become concrete.**
- Cache misses are slow because RAM access is 300× slower than register access
- String concatenation in a loop is O(n²) because each `+` allocates new memory
- Virtual calls are slower because they require an extra memory lookup (vtable)
- GC pauses happen because the collector needs to scan and compact heap memory

---

## 🛠️ Technologies Used

| Project | Language |
|---------|----------|
| Projects 1-5 | HDL (Hardware Description Language) |
| Project 6 | Python |
| Projects 7-8 | Python |
| Projects 9, 12 | Jack |
| Projects 10-11 | Python |

---

## 📚 Resources

- 📖 [The Elements of Computing Systems](https://www.nand2tetris.org/book) — Nisan & Schocken
- 🌐 [Nand2Tetris Official Website](https://www.nand2tetris.org)
- 🎓 [Coursera Part I](https://www.coursera.org/learn/build-a-computer)
- 🎓 [Coursera Part II](https://www.coursera.org/learn/nand2tetris2)

---

## 👤 About

Built by **Aqil Hummatli** — Backend Developer at PASHA Insurance, Baku, Azerbaijan.

Background in Chemical Engineering → self-taught software developer → backend engineering with Java, C#, and .NET Core.

This project represents a deep dive into how computers actually work — from transistors to tetris.

[![GitHub](https://img.shields.io/badge/GitHub-agilhummatli-black?style=flat&logo=github)](https://github.com/agilhummatli)

---

*"What I cannot create, I do not understand."* — Richard Feynman

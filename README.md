# Chip8.js

A Chip-8 emulator written in JavaScript (Node.js).

> [Chip-8](https://en.wikipedia.org/wiki/CHIP-8) is a simple, interpreted, programming language which was first used on some do-it-yourself computer systems in the late 1970s and early 1980s.

## Concepts

Concepts I learned while writing this program:

- The base system: specifically base 2 (binary), base 10 (decimal), base 16 (hexadecimal), how they interact with each other and the concept of abstract numbers in programming
- Bits, nibbles, bytes, ASCII encoding, and big and little endian values
- Bitwise operators - AND (`&`), OR (`|`), XOR (`^`), left shift (`<<`), right shift (`>>`) and how to use them for masking, setting, and testing values
- Using the Node built-in file system ([fs](https://www.npmjs.com/package/fs))
- The concept of a raw data buffer and how to work with it, how to convert an 8-bit buffer to a 16-bit big endian array
- Writing and understanding a 8-bit and 16-bit hex dump
- How to disassemble and decode an opcode into instructions a CPU can use
- How a CPU can utilize memory, stack, program counters, stack pointers, memory addresses, and registers
- How a CPU implements fetch, decode, execute

Articles I wrote based on aforementioned concepts: 

- [Understanding Bits, Bytes, Bases, and Writing a Hex Dump in JavaScript (Node)](https://www.taniarascia.com/bits-bytes-bases-and-a-hex-dump-javascript/)
- In progress: bitwise operators, masking, testing, and setting values.

## Installation

> This guide assumes you already have [Node.js](https://nodejs.org/en/) and [Yarn](https://yarnpkg.com/en/) installed. 

The only dependency of Chip8.js is [jest](https://jestjs.io/) for testing. Run `yarn` to install.

```bash
yarn
```

## Instructions

Chip-8 compatible ROMs can be saved in the `roms/` directory. A copy of *Connect 4* is shipped with Chip8.js (at `roms/CONNECT4`) for example and testing purposes.

### Load ROM (`yarn start`)

Create a ROM buffer of a ROM and load the data into the CPU. Execute the program.

```bash
yarn start roms/<ROM>
```

### View hex dump (`yarn hexdump`)

View a 16-bit hex dump of a ROM. (View more information on [bits, bytes, bases, and hex dumps](https://www.taniarascia.com/bits-bytes-bases-and-a-hex-dump-javascript/)).

```bash
yarn hexdump roms/<ROM>
```

The output will look something like this (using `CONNECT4` as an example).

```bash
000000 121a 434f 4e4e 4543 5434 2062 7920 4461
000010 7669 6420 5749 4e54 4552 a2bb f665 a2b4
000020 f655 6900 6801 6b00 6d0f 6e1f a2a5 600d
...
```

## Automated Testing

The unit tests for Chip8.js use the Jest testing framework. You can run all test suites with or without displaying coverage.

```bash
# Run test suites
yarn test

# Run test suites and view coverage
yarn test --coverage
```

Chip8.js has two suites of unit tests:

- Opcode instruction masks and arguments
- CPU implementation of instructions

### Instruction tests

The [instruction tests](tests/instructions.test.js) cover the `INSTRUCTION_SET` found in `constants/instructionSet.js`. Each instruction has:

- A `key`: for internal use
- An `id`: for a unique name
- A `name`: for the type of instruction)
- A `mask`: to filter out arguments from instruction signifiers)
- A `pattern`: to match the mask to the specific instruction pattern
- `arguments`, each of which contain:
  - A `mask`: to filter the nibble(s) to arguments
  - A `shift`: to shift it by location
  - A `type`: to signify the type of argument

#### constants/instructionSet.js (instruction 06)

```js
{
  key: 6,
  id: 'SE_VX_NN',
  name: 'SE',
  mask: 0xf000,
  pattern: 0x3000,
  arguments: [{ mask: 0x0f00, shift: 8, type: 'R' }, { mask: 0x00ff, shift: 0, type: 'NN' }],
}
```

Each unit test checks an opcode to an instruction and tests:

- The unique `id` to ensure the correct instruction is running for the mask/pattern
- The number of arguments
- The value of the arguments

#### tests/instructions.test.js (test 06)

```js
test('test instruction 06: 3xkk - SE Vx, byte', () => {
  expect(Disassembler.disassemble(0x3abb).instruction).toHaveProperty('id', 'SE_VX_NN')
  expect(Disassembler.disassemble(0x3abb).args).toHaveLength(2)
  expect(Disassembler.disassemble(0x3abb).args[0]).toBe(0xa)
  expect(Disassembler.disassemble(0x3abb).args[1]).toBe(0xbb)
})
```

There are 35 instruction tests for 35 opcodes (the first instruction, `CLS`, is no longer implemented).

### CPU tests

The CPU decodes the opcode and returns the instruction object from `constants/instructionSet.js`. Each instruction performs a specific, unique action in the `case`. The [CPU tests](tests/cpu.test.js) test the state of the CPU after an executing an instruction.

In the below example, the instruction is skipping an instruction if `Vx === kk`, otherwise it's going to the next instruction as usual.

#### classes/CPU.js (instruction 06)

```js
case 'SE_VX_NN':
  // Skip next instruction if Vx = kk.
  if (this.registers[args[0]] === args[1]) {
    this._skipInstruction()
  } else {
    this._nextInstruction()
  }
  break
```

Each CPU test:

- Loads a `RomBuffer` containing the data of a single opcode
- Sets up the state to make the instruction testable (if necessary)
- Executes the `step` method
- Tests all possible outcomes of an instruction and state updates

In this example, the instruction can either be skipped or not skipped depending on the arguments, and both cases are tested.

#### tests/cpu.test.js (test 06)

```js
test('test cpu 06: 3xkk - SE Vx, byte', () => {
  cpu.load({ data: [0x3abb] })
  cpu.step()

  expect(cpu.PC).toBe(0x202)

  cpu.load({ data: [0x3abb] })
  cpu.registers[0xa] = 0xbb
  cpu.step()

  expect(cpu.PC).toBe(0x204)
})
```

## Todo List

- [ ] Check for halts, throw errors and reset
- [ ] Begin I/O

## Acknowledgements

- Inspiration, guidance, and mentorship from [Vanya Sergeev](https://sergeev.io)
- [Cowgod's Chip-8 Technical Reference](http://devernay.free.fr/hacks/chip8/C8TECH10.HTM#8xy3), made by Thomas P. Greene
- [CHIP-8 - Wikipedia](https://en.wikipedia.org/wiki/CHIP-8)

## License

The code is open source and available under the [MIT License](LICENSE).
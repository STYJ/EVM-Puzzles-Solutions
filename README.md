# Solutions

A walkthrough of the solutions for each puzzle. Kudos to [Franco](https://twitter.com/fvictorio_nan) for these brain exercises!

# Tips

- Search on evm.codes for anything you do not know.
- Use evm.codes playground to get a better feel for what's going on
- Write it out on paper and walk through. Remember the EVM is stack based so FILO (or LOFI whichever you recognise).


## Puzzle 1

The first instruction is `CALLVALUE` which gets `msg.value` and pushes it onto the stack. The next instruction is `JUMP` which takes 1 input from the stack (which in this case is `CALLVALUE`) and jumps to that location. You can only jump to where there exists a `JUMPDEST` opcode. In this puzzle, `JUMPDEST` is at instruction 8 so that is where we want to go. The solution to this puzzle is 8.

## Puzzle 2

Puzzle 2 introduces 2 new opcodes `CODESIZE` and `SUB`. `CODESIZE` gets you the size of the code where each instruction is 1 byte and `SUB` is just subtract. Walking through the instruction, we're tasked to solve this mathematical equation `SUB(10, CODESIZE) = 6`. We can rearrange the formula to solve for `CODESIZE` i.e. 4. The solution to this puzzle is 4. 

## Puzzle 3

If you understood puzzles 1 and 2, puzzle 3 should be a piece of cake :) If you didn't understand puzzles 1 and 2, go and read the walkthroughs again. The solution to this puzzle is anything of length 4 bytes e.g. 0x00000000 or 0xffffffff or 0x12345678

## Puzzle 4

Similar to puzzle 2, we're tasked to solve this math equation `XOR(CODESIZE, CALLVALUE)`. `XOR` stands for [exclusive or](https://en.wikipedia.org/wiki/Exclusive_or). We need a value for `CALLVALUE` such that the output of `XOR` returns 10 since that is where `JUMPDEST` is located. `XOR` is a bitwise operator so we need to convert `CODESIZE` and `JUMPDEST` into their respective binary form. `CODESIZE` has a value of 12 which is 1100 in binary. `JUMPDEST` has a value of 10 which is `1010` in binary. The equation we need to solve is this `XOR(1100, CALLVALUE) = 1010`. `CALLVALUE` therefore needs to be 6 or 0110 for the `XOR` operation to return 10. The solution to this puzzle is 6.

## Puzzle 5

There's nothing new here that you don't already know. The `MUL` opcode stands for multiply. We need this equation `EQ(CALLVALUE * CALLVALUE, 0x0100)` to return 1. 0x0100 has a value of 256 so sqrt of 256 is 16! The solution to this puzzle is 16.

## Puzzle 6

`CALLDATALOAD` takes an offset to load 32 bytes from. Our offset is 0 (from `PUSH1 00`) so we just need `CALLDATA` to be 32 bytes with the last byte being `0a` (the location of `JUMPDEST`). The solution to this puzzle is 0x000000000000000000000000000000000000000000000000000000000000000a

## Puzzle 7

This is the first tough puzzle. Let's walk through this together step by step.

| Instruction | NAME             | STACK                          | MEMORY     |
|-------------|------------------|--------------------------------|------------|
| 00          | CALLDATASIZE     | CALLDATASIZE                   |            |
| 01          | PUSH1 00         | CALLDATASIZE <br /> 0          |            |
| 03          | DUP1             | CALLDATASIZE <br /> 0 <br /> 0 |            |
| 04          | CALLDATACOPY     |                                | CALLDATA   |
| 05          | CALLDATASIZE     | CALLDATASIZE                   | CALLDATA   |
| 06          | PUSH1 00         | CALLDATASIZE <br /> 0          | CALLDATA   |
| 08          | PUSH1 00         | CALLDATASIZE <br /> 0 <br /> 0 | CALLDATA   |
| 0A          | CREATE           | ADDR                           | CALLDATA   |
| 0B          | EXTCODESIZE      | EXTCODESIZE                    | CALLDATA   |
| 0C          | PUSH1 01         | EXTCODESIZE  <br /> 1          | CALLDATA   |
| 0E          | EQ               | EQ(1, EXTCODESIZE)             | CALLDATA   |
| 0F          | PUSH1 13         | EQ(1, EXTCODESIZE) <br /> 13   | CALLDATA   |
cont...

Looking at the instructions, we need to solve for `EQ(1, EXTCODESIZE)`. If we can somehow get `EXTCODESIZE` to be 1, this `EQ` comparison will return 1 thus allowing us to jump to location 13. We know that we want a series of opcodes such that after executing the instructions, it returns something of 1 byte. In order to return anything, we need to use the `RETURN` opcode. This opcode takes the top 2 values on the stack as offset O and size S. It will proceed to return S amount of bytes **in memory** after offset O.

Since we only want to return 1 byte, we will use a size of 1. We can also use an offset of 0 because the scratch space (first 64 bytes) is big enough for this. The last 5 bytes of our calldata should be `60016000F3`. Our memory is still empty so we need some opcodes to move stuff from the stack to memory i.e. `60<1 byte>600052`. Putting these 2 sets of instructions together, we get `60<1 byte>60005260016000f3`. A solution to this puzzle is `60<1 byte>60005260016000f3` e.g. `60ff60005260016000f3`.

If you didn't fully understand this explanation, play around with the playground examples for `CREATE` and `EXTCODESIZE` before re-reading this again.

## Puzzle 8

Puzzle 8 is very similar to puzzle 7. The goal of this level is to deploy a contract such that if you tried to do a plain `CALL` on it, it needs to revert i.e. the return value of `CALL` is 0. This level is a bit more tricky because you need to understand how numbers are padded when added to memory. Try storing the value 1 into memory and see what you get! Like puzzle 7, there are many solutions to this level so here is one possible solution `0x646003565BFD6000526005601BF3`. At a high level, this solution can be broken down to the following:

| NAME                                                      |
|-----------------------------------------------------------|
| PUSHX CONTRACT TO PUSH IN OPCODES                         |
| PUSH1 00                                                  |
| MSTORE                                                    |
| PUSHY SIZE OF CONTRACT                                    |
| PUSHZ OFFSET TO RETRIEVE ONLY THE CONTRACT FROM MEMORY    |
| RETURN                                                    |

In my solution, my contract is `6003565BFE`. This has 5 bytes so 

| OPCODES         | NAME                        |
|-----------------|-----------------------------|
| 646003565BFD    | PUSH5 6003565BFE (contract) |
| 6000            | PUSH1 00                    |
| 52              | MSTORE                      |
| 6005            | PUSH1 05 (size)             |
| 601B            | PUSH1 27 (offset)           |
| F3              | RETURN                      |

You can break `6003565BFE` down to

| OPCODES        | NAME     |
|----------------|----------|
| 6003           | PUSH1 03 |
| 56             | JUMP     |
| 5B             | JUMPDEST |
| FD             | REVERT   |

which basically does nothing but revert.

A solution to this puzzle is `0x646003565BFD6000526005601BF3`

## Puzzle 9

Puzzle 7 and 8 are the hardest so if you can complete those 2, the rest should be a piece of cake! Like the previous challenges, walk through the opcodes to understand what you're looking for. We're solving for `CALLDATA` and `CALLVALUE` such that `LT(3, CALLDATASIZE)` and `CALLDATA` * `CALLVALUE` = 8. The solutions to this puzzle are either 1) calldatasize of 4 and a value of 2 e.g. 0xffffffff, 2 or 2) calldatasize of 8 and a value of 1 e.g. 0xffffffffffffffff, 1.

## Puzzle 10

We're trying to solve for `CALLVALUE` such that `ADD(CALLVALUE, 0x0a)` gets you 0x19 (decimal for 26). We know that 0x0a is 11 so 26 - 11 = 15 i.e. `CALLVALUE` is 15 or 0xff. You also need to figure out a value for `CALLDATASIZE` such that `MOD(CALLDATASIZE, 3)` returns 0 so any multiple of 3 will do. The solutions to this puzzle are callvalue of 15 and calldata whose length is a multiple of 3 e.g. 0xffffff.

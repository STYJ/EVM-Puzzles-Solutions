# Solutions

A walkthrough of the solutions for each puzzle. Kudos to [Franco](https://twitter.com/fvictorio_nan) for these brain exercises!

# Tips

- Search on evm.codes for anything you do not know.
- Use evm.codes playground to get a better feel for what's going on
- Write it out on paper and walk through. Remember the EVM is stack based so FILO (or LOFI whichever you recognise).


## Puzzle 1

The first instruction is `CALLVALUE` which gets `msg.value` and pushes it onto the stack. The next instruction is `JUMP` which takes 1 input from the stack (which in this case is `CALLVALUE`) and jumps to that location. You can only jump to where there exists a `JUMPDEST` opcode. In this puzzle, `JUMPDEST` is at instruction 8 so that is where we want to go. The solution is therefore 8.

## Puzzle 2

each instruction 1 byte
10 instructions so 10 bytes or 320 bits
you want sub(10, something) = 6 since that's where jumpdest is at so answer = 4

## Puzzle 3

calldatasize gets you size of calldata
0x00 = size of 1
something jump, location of jumpdest = instruction 4
solution = 0x00000000 or any 8 hexidecimal characters

## Puzzle 4

xor(codesize, callvalue)

codesize = 12 = 1100
jumpdest = 10 = 1010
callvalue should be 6 or 0110
xor(1100, 0110) = 1010

## Puzzle 5

1. callvalue
2. callvalue, callvalue
3. callvalue^2
4. callvalue^2, 0x0100
5. eq(callvalue^2, 0x0100)
location alr specified as 0c. jumpi is conditional jump only if b is true
i.e. eq needs to return 1. 0x0100 = 256 so callvalue should be 16 as 16^2 = 256.

## Puzzle 6

Want to load 32 bytes from offset of 0 in calldata
solution = 0x000000000000000000000000000000000000000000000000000000000000000a

## Puzzle 7

This was pretty tough. Had to compare the playground examples for create vs extcodesize on evm.codes.

I noticed 2 diff

1) the first byte is diff
2) there's a substring that looks very similar but also has 1 byte diff

create = 0x63FFFFFFFF60005260046000F3
extcodesize = 0x7fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff60005260206000f3

0x7f = 127
0x63 = 99

0x04 = 4 bytes
0x20 = 32 bytes

since diff is 27 bytes, 127-99 is also 28 bytes. I figured maybe my first byte should be 0x60 = 96 and I should use 0x01 i.e.

create = 63<any 4 bytes>60005260 04 6000F3
extcodesize = 7f<any 64 bytes>60005260 20 6000f3
solution = 60<any 1 byte>60005260 01 6000f3

these are opcodes btw.

push size onto memory and return

## Puzzle 8

Similar to puzzle 7, walk through it. The goal is to deploy a contract that if you tried to do a plain .call on it, it needs to revert so that the output of .call is 0.

0x67600035600757FE5B60005260086018F3
0x646003565BFD6000526005601BF3

0x646003565BFD  PUSH5 6003565BFE
6000            PUSH1 00
52              MSTORe
6005            PUSH1 05
601B            PUSH1 27
F3              RETURN


The contract does this

6003            PUSH1 03
56              JUMP
5B              JUMPDEST
FD              REVERT

Which basically does nothing but revert. I tried purely revert only e.g. 0x6160FD6000526002601DF3 or 0x60FD6000526001601EF3 but it wouldn't pass

## Puzzle 9

Really easy compared to 7 and 8. Walk through the opcodes, basically you want to specify something for calldata such that lt(3, calldatasize) and callvalue such that calldata * callvalue = 8. The only solution to this is either calldatasize = 4 and value = 2 or calldatasize = 8 and value = 1

0xffffffff, 2
0xffffffffffffffff, 1

## Puzzle 10


Similar to 9. Just walk throughh the opcodes. You'll realise you need to specify callvalue such that add(callvalue, 0a) gets you 0x19 i.e. callvalue = 15 or 0xff. You also need to figure out a value for calldatasize such that mod(calldatasize, 3) returns 0 so any multiple of 3 will do. 

callvalue 15
calldata 0xffffff

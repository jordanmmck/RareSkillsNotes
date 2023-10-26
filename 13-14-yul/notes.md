# Yul

so when we write yul we use an `assembly` block inside solidity. and it's sort of like writing opcodes right in that block.

huff is way lower level tho. in huff we interact with a stack. and memory etc directly. huff is actual bytecode with a few nice formatting things basically.

yul has for-loops. that's like, massively higher level than huff where you have to create the loop yourself and manage it all via the damn stack.

## info

```js
assembly {
    let myvar;
    for {let i := 0} lt(i, lim) { i := add(i, 1) } {
        // do stuff
    }
    slot := myvar.slot
    offset := myvar.offset
    let value := sload(myvar.slot)
    let shifted := shr(mul(myvar.offset, 8), value)
    let e := and(0xfffffffff, shifted)
}
```

## memory

memory is just an array. you can write in 8 byte chunks, but must read 32 bytes chunks. then mask or w/e.

need memory if:

- return value to external calls
- set f'n args for external calls
- get values from external calls
- revert with error string
- log messages
- create other smart contract
- use keccak256

## inter-contract calls

```js
contract CalldataDemo {
        fallback(bytes calldata data) external returns (bytes memory returnData) {
                assembly {
                        let cd := calldataload(0)
                        let selector := shr(0xe0, cd)

                        switch selector
                        case 0xabc123ee {
                                returnUint(2)
                        }
                        case 0x12345678 {
                                returnUint(getNotSoSecretValue())
                        }
                }
        }
}
```

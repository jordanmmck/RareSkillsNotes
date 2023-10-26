# Questions

- figure out how to write bytes literal. ie. `bytes32(0)` or whatever. all those literal ways to write out hex data or any data.
- diamond pattern and other "meta" patterns
- try / catch
- the memory keyword
- forge console vs. console2

- suppose we have an address written into our bytecode as a constant. how does the EVM actually know how to deal with that length of data? like, the bytecode is a string of bytes right. most of those are like 0x80 or 0x40. but what if i have 0x80123456789abcdef? how does it know i'm pushing an address onto the stack and not just a single byte or w/e? i think maybe we have PUSH32 etc. yup.

- evm puzzle # 7. wtf.

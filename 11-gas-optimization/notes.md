# Gas Optimization

- init code will be shorter if your constructor is payable...

## contract creation

<init code> <runtime code> <constructor parameters>

- compiler will append a bunch of metadata to end of compiled bytecode
- constructor params go at the end of compiled bytecode

## store packing

storage vars are laid out in order of inheritance.

also, vars are laid out least significant bit first kinda way. ie.

```js
uint8 a = 1;
uint8 b = 2;
```

000000000...2000000000000...1

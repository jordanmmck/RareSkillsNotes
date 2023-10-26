# Notes

remember, contracts each have their own state. each is a little world. when we call to another contract we (generally) do not have access to the state of the original contract. we jump over to the new world and only data sent along explicitly in the call is known to the called contract. the exception of course is `delegatecall` where we can "use" the _code_ of another contract to act upon our contract's state. and in that case we have NO access to the called contract's state. we just "borrow" it's code. generally, `msg` is not changed as well with `delegatecall`. so `msg.sender` in a delegatecall should still be the original caller, maybe an EOA, and not the calling contract's address as it would be in a normal `call`.

`external`: you can _still_ call a function that is marked `external` from within the contract by using `call`. but i suppose it would spawn a new EVM and act just like any other call?

`constant` and `immutable`: `constant` must be set at compile time. truly hardcoded. `immutable` is set at constructor time. so if you wanted to have a contract which definitely self-destructs or pauses forever after 10,000 epochs, then you could set an `immutable` var `deathEpoch` which would be set at construction time as block.epoch + 10,000 or w/e. any variable that _is_ constant/immutable should be set as such. maximum restriction is good.

when a contract which inherits from ERC20 or whatever else is deployed, _all_ that code is deployed as one contract. you might have 4 levels of inheritance, but it's all flattened. it's all just internal calls.

you can change visibility when you override a function. but only to more restrictive things.

`libraries` are like contract but deployed only once at a specific address, then called using delegatecall. so it's kinda like we have our NPM ecosystem of libraries onchain. they're just there. a thousand different contract can just delegatecall to some math.solv1.12.1 library that does w/e.

`using` `for`. ie. `using A for B`. basically we can attach functions to B. `using Address for address` tacks on some extra function to `address`.

## modifiers

inside a modifier you may have:

```
modifier A() {
    require();
    _;
    require();
}
```

so the first require runs before your function you are modifying, then the function runs, then the second require executes. so the `_` basically means "call the function you are modifying".

## structs

If you have a mapping of structs and the struct has a mapping inside it, then `delete` on a given item in the parent mapping won't actually delete the child mapping!

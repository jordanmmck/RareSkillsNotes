# Week 7 CTFs

## Broad Security Issues

- Money stolen
- Money frozen
- Insufficient rewards
- Excessive rewards

## ERC20 Snapshot

## Specific Security Issues

**Reentrancy**
When contract calls out to another contract. This can include sending ether to some contract. Or transferring tokens. Once the other contract has control it can do whatever it wants.

**Access Control**
Access must be restricted properly.

**Improper Input Validation**
Acess control controls who calls a function, input validation controls what it can be called with. Use require statements.

**Excessive function restriction**
Having too tight of restrictions could result in funds not being stolen, but locked forever!

**Double voting or msg.sender spoofing**
Be careful how voting is recorded/measured. Token voting can be exploited if a user can vote then transfer tokens to another address they control and vote again.

**Flashloan Governance Attacks**
Don't forget flashloans! Can a user just borrow a zillion coins, vote, then pay back the coins?

**Flashloan Price Attacks**
Take flash loan to distort some price oracle and liquidate something or whatever.

**Bypassing the contract check**
Avoid checking if an account is a smart contract. it's an anti-pattern.

**tx.origin**
Avoid using it.

**Gas Griefing or Denial of Service**
Using lots of gas in a way that somehow annoys some contract?

**Insecure Randomness**
Use RANDAO.

**Using Chainlink Randomness Oracle Wrong**
Read their docs.

**Getting stale data from a price Oracle**
Read their docs.

**Relying on only one oracle**
Use multiple oracles

**Oracles in general are hard to get right**
Avoid oracles tbh.

**Mixed accounting**
Use discrete accountig, not contract balance. Even if you don't have a receive or fallback, someone can still self destruct a contract and force ether to your contract.

**Treating cryptographic proofs like passwords**
You must check msg.sender. Just because the tx sender has some correct info like the leaf of a merkle tree doesn't mean anything. You must tie it back to their address.

**Solidity does not upcast to the final uint size**
This code will revert:

```js
function limitedMultiply(uint8 a, uint8 b) public pure returns (uint256 product) {
    product = a * b;
}
```

We need to cast a and b to uint256 in the function body.

**Solidity sneakily makes some literals uint8**
The following reverts because the ternary op returns a uint8

```js
function result(bool inp) external pure returns (uint256) {
    return uint256(255) + (inp ? 1 : 0);
}
```

Fix:

```js
function result(bool inp) external pure returns (uint256) {
    return uint256(255) + (inp ? uint256(1) : uint256(0));
}
```

**Solidity downcasting does not revert on overflow**
Use SafeCast lib.

**Writes to storage pointers don’t save new data**
The code looks like it copies the data in myArray[1] to myArray[2] but it doesn't.

```js
contract DoesNotWrite {
    struct Foo {
        uint256 bar;
    }
    Foo[] public myArray;

    function moveToSlot0() external {
        Foo storage foo = myArray[0];
        foo = myArray[1]; // myArray[0] is unchanged
        // we do this to make the function a state
        // changing operation
        // and silence the compiler warning
        myArray[1] = Foo({bar: 100});
    }
}
```

**Deleting structs that contain dynamic datatypes does not delete the dynamic data**
If a mapping or dynamic array is inside a struct, deleting the struct will not delete the mapping. With the exception of deleting an array, the delete keyword can only delete one storage slot.

**ERC20: Fee on transfer**
You can't rely on a transfer of an arbitrary token transferring exactly the expected amount. It may have a fee.

**ERC20: rebasing tokens**
Beware of rebasing tokens. They may fuck up your logic.

**ERC20: Not all ERC20 tokens return true**
Beware.

**Unchecked return values**
A smart contract can be called by calling the function with the interface definition, or by using the .call method. If you call via .call your code will not revert if the contract you called reverts! So you may want to check a return value or something.

**msg.value in a loop**
Danger. Sender may be able to re-use msg.value.

**Try Catch is hard to get right**
Cannot distinguish between panic and revert. Avoid.

**Insecure Delegate Call**
Hands over all control to delegate. Don't use unless you trust!

**Upgrade bugs related to proxies**

- selfdestruct and delegatecall should not be used inside implementation contracts
- care must be taken that storage variables never overwrite each other during upgrades
- calling external libraries should be avoided in implementation contracts because it isn’t possible to predict how they will affect storage access
- deployer must never neglect to call the initialization function
- not including a gap variable in base contracts to prevent storage collision when new variables are added to the base contract (this is handled by the hardhat plugin automatically)
- the values in immutable variables are not preserved between upgrades
- doing anything in the constructor is highly discouraged because future upgrades would have to carry out identical constructor logic to maintain compatibility.

**Overpowered Admins**
Give minimal power to admins and never forget their keys could be hacked!

**Use Ownable2Step instead of Ownable**
Using plain Ownable can result in loss of contract if you fuck up.

**Rounding Errors**
You have to round. Be smart about it. Divide last.

**Frontrunning**
Beware.

**Signature Related**
Signatures can used and verified within contract but outside the normal tx context.

- Use openzeppelin's library to prevent malleability attacks and recover to zero issues
- Don't use signatures as a password. The messages needs to contain information that attackers cannot easily re-use (e.g. msg.sender)
- Hash what you are signing on-chain
- Use a nonce to prevent replay attacks. Better yet, follow EIP712 so that usese can see what they are signing and you can prevent signatures from being re-used between contracts and different chains.

**Assuming smart contracts are immutable**
Don't assume some contract that isn't yours is immutable. It might upgrade.

**Transfer() and send() can break with multi-signature wallets**
Don't use these.

**Is Arithmetic overflow still relevant?**
Don't use safemath. Use 0.8+.

**What about block.timestamp?**
Avoid.

**Corner Cases, Edge Cases, and Off By One Errors**
off by one. etc.

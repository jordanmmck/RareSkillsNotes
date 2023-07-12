# SafeERC20

> Why does the SafeERC20 program exist and when should it be used?

## transfer & transferFrom

SafeERC20 is a wrapper around some ERC20 operations created by OpenZeppelin that attemps to mitigate potential security issues and other pitfalls that can arise around standard ERC20 operations.

ERC20 is a token interface standard. What that means is that we have defined a certain set of function interfaces and event signatures that we would like to see implemented on tokens calling themselves "ERC20 tokens". But there isn't actually anything *enforcing* strict adherence to this standard. Solidity developers are free to deploy contracts with any set of functions and events they wish.

So, there are token contracts running on Ethereum which *mostly* adhere to ERC20, but which may deviate in small ways. For example, according to ERC20, the methods `transfer` and `transferFrom` should always return a boolean. But some token implementations are not reliable in this sense — they may revert, or fail to return any value, or otherwise fail to return true or false as expected. This deviation away from the ERC20 spec may cause problems in our contract if we're writing logic which expects adherence to the ERC20 spec. And, of course we don't want to have to audit the code of all the possible tokens our contract may interact with to make sure we only interact with "true" ERC20 contracts.

The SafeERC20 handles this potential variation in behavior for us. Specifically it wraps certain calls to ERC20 methods in functions which revert if the called method fails in any way.

## allowance double spend

ERC20 tokens have the notion of "allowance". This functionality allows token holders to optionally grant permission for another address to spend, or take, some of their tokens.

The way this allowance is set is that the token holder calls `allowance` on the contract and specifies the recipient address and some maximum amount that the recipient can take.

There is a potential for a sort of double-spend attack to occur given this situation.

Suppose Alice has set some allowance amount to Bob of 100 tokens. Alice then decides to change the allowance to 90 tokens. Bob can potentially "spend" the 100 tokens before Alice's change is mined, and then spend another 90 once her transaction completes. So, Alice had intended for Bob to only receive a maximum of 90 tokens, but in the end he has taken 190.

This attack is possible because the contract storage which keeps track of allowance does not have "long-term memory". When a new allowance is set the contract does not "know" that some amount of the previous allowance was already used.

A mitigation to this attack is to first set the allowance to zero (and ensuring this change completes successfully), before setting it to another value.

SafeERC20 provides `increaseAllowance` and `decreaseAllowance` which are not susceptible to this attack.

## contract verification

There are certain problems with other wrapper libraries whereby a call to `SafeTransfer` may result in unexpected behavior if the address that is passed to the call is *not* a smart contract, but instead is an EOA. SafeERC20 protects against this by checking the code size of the account passed to this function. If the code size is zero, then the account is an EOA, and the call will revert. 

## When to use SafeERC20

It would seem that using SafeERC20 would be a good idea in most situations where your contract is interacting with ERC20 contracts. I can't think of many reasons not to use it, except perhaps if we are well aware of all possible issues and are certain that they are not relevant for our contract.

# Testing Notes

## Slither

> Slither is a Solidity static analysis framework written in Python3. It runs a suite of vulnerability detectors, prints visual information about contract details, and provides an API to easily write custom analyses. Slither enables developers to find vulnerabilities, enhance their code comprehension, and quickly prototype custom analyses.

Slither is a software package which essentially looks for a ton of common errors in our contracts. It will attempt to find reentrancy bugs, violations to naming conventions, etc.

## Mutation Testing

Mutation testing is a way of testing the quality of the tests that we've written.

In mutation testing we have a program modify our actual contract source code and then check to see if our tests pass or fail. If our tests are well written, these changes to our code should tend to be caught by our tests.

Each mutation is called a "mutant". If the mutant is "caught" by our tests then we consider it killed. If a mutated version of our program is _not_ caught by our tests then that mutant has not been caught, suggesting that perhaps our tests are not thorough enough.

## gasleft

Solidity has a `gasleft()` function which returns the remaining gas during a contract call. This function can be used to prevent a transaction from reverting due to running out of gas.

An example of where this might be used could be if our contract is looping over a list of addresses and doing some operation for each address. Maybe the contract sends some ETH to each address, or updates some storage, or something else.

If we just create the tx and start the contract looping over the list without any checks, then the tx may run out of gas after successfully "finishing" operations for many addresses. Then, despite the fact that those operations executed properly, the whole tx may revert if we run out of gas on the Nth operation.

We may instead want to loop over as many addresses as we can given the gas limit for the tx, and make sure we stop before running out of gas. Any items in our list that were not handled, can perhaps be dealt with in another tx.

`gasleft` can also be used to measure gas use for testing purposes. We can check the remaining gas before some section of code, then check it again after, and compute the difference. This tells us how much gas the intermediate code used.

Note: the opcode behind `gasleft` itself costs 2 gas to execute.

## Testing Internal Functions

To test an internal function create a contract which inherits this contract and which exposes that internal function and external and therefore callable.

Another approach would be to _modify_ the function on our actual contract to make it external, and therefore callable, but this is **not** a good idea. We want to test our contract exactly as we expect it to be deployed. We don't want to alter it for testing purposes then change it back and deploy.

It seems to me that this "harness" approach is not quite ideal either though. Because, in creating a harness and calling to it, we're still not quite testing the "natural" state or situation that we expect or want. We are creating an entirely different contract for the sole reason of gaining access to some internal state.

The thing we want to do, is to be able to go "inside" the contract and call functions which definitely should not, and will not, be called from outside. But, maybe it would be better if we had a testing tool which _did_ in fact allow us to go inside the contract and tinker with it. It seems it would be useful to be able to instrument the contract in a deep way, such that we can view all internals operations and function return values, and possible combinations of states and so on.

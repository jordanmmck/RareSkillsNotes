# Wrapped NFT

## Q1

> Besides the examples listed in the code and the reading, what might the wrapped NFT pattern be used for?

Wrapping tokens or NFTs can essentially be a way to "convert" a token or NFT to an entirely separate contract. So, we can have completely new functionality, potentially different supply, different contract owners/admin. For example, we might want to migrate away from a contract which is controlled by some small set of admins and move toward a DAO controlled system.

Wrapping can provide all kinds of special functionality that is beyond ERC721 or ERC20 scope as well. We could imagine wrapping an NFT such that it maintains a list of previous owners. Or some "basic" NFT could be wrapped in order to gain extra properties which allow it to be used in a card game of sorts. The penguins NFTs could be wrapped to have RPG-esque stats like strength, defense, etc. You could even have stateful properties like HP.
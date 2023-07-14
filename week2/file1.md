# ERC721A

## Q1

> How does ERC721A save gas?

[blog post](https://www.azuki.com/erc721a)

The **first optimization** is in removing duplicate storage from OpenZeppelin's ERC721Enumerable implementation.

I don't see where this happening though. The ERC721Enumerable.sol file has no mention of `metadata`. To find metadata we have to go down to the ERC721.sol file. There we can see `name` and `symbol` for the collection, and then `tokenURI` for the individual NFT.

[?]

The **second optimization** is updating owner's balance once per batch mint request instead of per minted NFT.

The ERC721.sol file from OZ has a mapping from owner address to token count. When an NFT is minted the count is incremented for that user. For batch minting it is more gas efficient to simply update that count *once*, from 2 directly to 7 for example, rather than 5 times from 2 to 3, 3 to 4, etc.

The **third optimization** is in updating the owner data once per batch mint request, instead of per minted NFT.

ERC721 has a mapping from tokenID to address. This is the central data structure which tracks token ownership. In the OZ ERC721 implementation a user who owns 20 NFTs will have their address appear in this mapping 20 times. 

The optimization here is that we can save on storage writes by writing that address just *once* in the case that their 20 NFTs are contiguous in the mapping. Basically we write the owner address at the index of the *first* NFT they own, then for any contiguous subsequent NFTs they own we leave the address blank.

## Q2

> Where does it add cost?

The first place these optimizations *add* cost is in the cotract size. All of these changes require extra code complexity and the associated gas cost there must be shouldered by the project upon deploy time.

Some of these optimziations also involve more complexity in terms of calls to the contracts made by users. In the second optimization for example, there may be some small extra complexity added to ensure that the user's NFT count is only updated once. So in the case that they only mint *one* NFT, they may be slightly worse off than if the contract were not optimized to benefit batch mints.

The third optimization adds non-trivial complexity and will likely result in higher costs after the initial minting occurs. As users transfer NFTs between each other, there will be higher cost to updating the `_owners` data stucture now that it has more complex logic.

## Q3

> Why shouldn’t ERC721A enumerable’s implementation be used on-chain?

The changes made here make some trade-offs to reduce gas costs for initial minting. Perhaps projects which intend to survive long-term should consider how ongoing gas costs will compare to those saved during minting. in the case of cryptopunks for example it might not have been smart to save gas on minting only to make transfers and other operations more expensive later on, because the total initial mint cost is probably lower now than the total cost incurred via transfers etc.

# Events

> Revisit the solidity events tutorial. How can OpenSea quickly determine which NFTs an address owns if most NFTs don’t use ERC721 enumerable? Explain how you would accomplish this if you were creating an NFT marketplace.

The primary way in which NFT ownership can be tracked is through events. ERC721 dictates that whenever an NFT is transferred or minted (transferred from the zero-address) an event is emitted detailing the relevant info.

Supposing that we have a system which has records of ownership for some NFT that are accurate up to some point in time, we can then easily keep our records up-to-date from there by simply listening for events in each block. Whenever one of the NFTs is transferred or minted we update our records. 

Suppose however that we *do not* have up-to-date records for some NFT collection of interest. We then may have to go back through ethereum history to the point where the minting begins and collect all relevant events from that point forward until we get caught up to the present.

If I were building an NFT marketplace, I think the approach I would take to this problem would be to maintain an internal, offchain database which tracks all major NFT collections, and which listens for events and thereby keeps all collections up-to-date. Having all this data in a fast offchain DB would allow for fast queries and quick responses on the front-end, and should also be easy to keep up-to-date. The amount of data being tracked, even for thousands of NFT collections, really isn't that much. The image files themselves would likely take up multiples more data than the ownership info and other metadata that is pulled from ethereum.
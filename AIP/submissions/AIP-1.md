---
layout: default
aip: 1
title: NFT Standard
status: Active
type: Core
author: Aventus Network <info@aventus.io>
created: 2022-04-05
---


## Abstract/Summary

This standard will introduce a new pallet described as the NFT-Manager pallet. This pallet, is designed to support the actions and properties of all NFTs. The NFT-Manager breaks the mould of contract-limiting NFTs empowering Aventus in supporting NFT natively i.e. built into the Blockchain infrastructure. Sending NFT from one account to another should now be as easy as sending AVT tokens. This standard will also allow for the native support of Royalties for NFTs.

## Motivation

The initial motivation for this standard was the desire and urgency to have NFT functionality on the Aventus network, and this is the primary reason we believe this standard should be implemented. With the introduction of this standard NFTs can be minted, sold and resold on the Aventus blockchain. Furthermore, as Aventus is compatible with Ethereum and soon Polkadot, these NFTs can migrate cross-platform increasing in value and accumulating a rich metadata. This standard will support NFT operations in use cases such as Gaming, DeFi, NFT marketplaces, etc.

## Specification

There are several properties that aid in the identification of an NFT such as the nft_id, origin, and unique_external_ref. The first two variables provide information about the NFT while the third provides a reference to the external location of the related NFT. The owner property of the NFT provides the provenance history of the NFT, ensuring that the ownership history of the NFT can be tracked. This provenance empowers the Blockchain to pay out Royalties.

```sh
fn mint_single_nft(
    origin,
    nft_id: NftId,
    unique_external_ref: Vec<u8>,
    owner: T::AccountId,
    royalties: Vec<Royalty>,
    minter_t1_address: H160)
```

The above function signature describes the functionality of minting batch NFT on the Aventus blockchain. There is just one addition to the function signature for the batch NFT which is the batch_id which identifies the batch of NFTs.

```sh
fn mint_batch_nft(
   origin,
   nft_id: NftId,
   batch_id: NftBatchId,
   unique_external_ref: Vec<u8>,
   owner: T::AccountId,
   royalties: Vec<Royalty>,
   minter_t1_address: H160,
   total_supply: u64)
```

The above function signature describes the functionality of minting batch NFT on the Aventus blockchain. There is just one addition to the function signature for the batch NFT which is the batch_id which identifies the batch of NFTs.

The two code snippets above are function signatures.

NFT details are understood as two structs, the first contains "external" properties of the NFT such as the owner address `AccountId`, the external reference `unique_external_ref`, etc.

```sh
pub struct Nft<AccountId: Member> {
    /// Unique identifier of a nft
    pub nft_id: NftId,
    /// Id of an info struct instance
    pub info_id: NftInfoId,
    /// Unique reference to the NFT asset stored off-chain
    pub unique_external_ref: Vec<u8>,
    /// Transfer nonce of this NFT
    pub nonce: u64,
    /// Owner account address of this NFT
    pub owner: AccountId,
    /// Flag to indicate if the vendor has marked this NFT as transferrable or not
    ///  - false: able to be transfered (default)
    ///  - true: not able to be transfered
    pub is_locked: bool,
}
```

The second struct defines the lower level info properties of the NFT such as the royalties `royalties` and total supply `total_supply`, etc.

```sh
pub struct NftInfo {
    /// Unique identifier of this information
    pub info_id: NftInfoId,
    /// Batch Id defined by client
    pub batch_id: Option<NftBatchId>,
    /// Royalties payment rate for the nft.
    pub royalties: Vec<Royalty>,
    /// Total supply of NFTs in this collection:
    ///  - 1: it is for a singleton
    ///  - >1: it is for a batch
    pub total_supply: u64,
    /// Minters tier 1 address
    pub t1_authority: H160,
}
```

When a new single NFT is minted, the `batch_id` in the return block below is set to None as the single NFT would not be part of a batch of NFTs.

```sh
pub fn new(info_id: NftInfoId, royalties: Vec<Royalty>, t1_authority: H160) -> Self {
    return NftInfo {
        info_id,
        batch_id: None,
        royalties,
        total_supply: 1u64,
        t1_authority,
    };
}
```

The code block below is a direct snippet from the pallet and the comments signified by the preceding “///” are there to provide you with a clearer insight into the workings of the line of code.

```sh
decl_storage! {
trait Store for Module<T: Trait> as NftManager {
/// A mapping between NFT Id and data
pub Nfts get(fn nfts): map hasher(blake2_128_concat) NftId => Nft<T::AccountId>;

        /// A mapping between NFT info Id and info data
        pub NftInfos get(fn nft_infos): map hasher(blake2_128_concat) NftInfoId =>         NftInfo;

        /// A mapping between the external batch id and its nft Ids
        pub NftBatches get(fn nft_batches): map hasher(blake2_128_concat) NftBatchId => Vec<NftId>;

        /// A mapping between the external batch id and its corresponding NtfInfoId
        pub BatchInfoId get(fn batch_info_id): map hasher(blake2_128_concat) NftBatchId => NftInfoId;

        /// A mapping between an AccountId and a flag to hold a list of approved minters
        pub ApprovedMinters get(fn approved_minters) : map hasher(blake2_128_concat) T::AccountId => bool;

        /// A mapping between an ExternalRef and a flag to show that an NFT has used it
        pub UsedExternalReferences get(fn is_external_ref_used) : map hasher(blake2_128_concat) Vec<u8> => bool;

        /// The Id that will be used when creating the new NftInfo record
        pub NextInfoId get(fn next_info_id): NftInfoId
    }
}
```

## Backwards Compatibility

N/A

## Tests

All tests can be found [HERE](https://github.com/Aventus-Network-Services/avn-node/tree/master/aventus-frame-pallets/nft-manager/src/tests)

## Reference Implementation

This pallet does not build on any previous NFT implementation in the Aventus ecosystem.

## Copyright

Copyright and related rights waived via [CC0](../../../LICENSE)

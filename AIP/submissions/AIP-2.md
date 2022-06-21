---
layout: default
aip: 2
title: Batch-NFT Standard
status: Draft
type: Core
author: Aventus Network <info@aventus.io>
created: 2022-06-10
---

## Abstract/Summary

This proposal introduces support for the minting and sale of Batch NFTs on the AvN.

## Motivation

This standard is designed and created to introduce support for the minting and sale of Batch NFTs on the AvN. This will support use cases and dApps that require minting and managing multiple instances of an NFT.

## Specification

This includes the creation of public endpoints to process batch sales. It is designed to be as generic as possible when dealing with batch nfts i.e. it doesn’t assume anything about how or where these nfts will be sold. Each batch can hold a single nft to an unlimited number of nfts.

Both `NftInfoId` and `NftBatchId` are `U256` linking to structs that define the properties of the NFT. The structure for minting batch NFTs is similar to that for minting a single NFT. When minting a single NFT, the `total_supply` is set to `1` but with minting batch NFTs,the `total_supply` determines the max number of instances that can be minted in a batch. Given that all data about a batch must be supplied at the batch creation stage, the `royalties` set apply to all nfts in the batch and is immutable.

The `creator` here is the AvN account that creates the batch. The `creator` retains ownership of that batch until all the NFTs are minted. When minting a single NFT, the `creator` is defined as `creator: None`.

```rust
pub fn create_batch<T: Config>(
    info_id: NftInfoId,
    batch_id: NftBatchId,
    royalties: Vec<Royalty>,
    total_supply: u64,
    t1_authority: H160,
    creator: T::AccountId)
{
    let info = NftInfo::new_batch(info_id, batch_id, royalties, t1_authority, total_supply, creator);
    BatchInfoId::insert(batch_id, &info.info_id);
    <NftInfos<T>>::insert(info.info_id, info);
}
```

The `t1_authority` is the address of a smart contract that manages the NFT, in this case, an Ethereum smart contract most likely linked to an NFT marketplace. This field cannot be left empty during batch creation.

```rust
/// Creates a new batch
#[weight = T::WeightInfo::proxy_signed_create_batch(MAX_NUMBER_OF_ROYALTIES)]
fn signed_create_batch(origin,
    proof: Proof<T::Signature, T::AccountId>,
    total_supply: u64,
    royalties: Vec<Royalty>,
    t1_authority: H160) -> DispatchResult
{
    let sender = ensure_signed(origin)?;
    ensure!(sender == proof.signer, Error::<T>::SenderIsNotSigner);
    ensure!(t1_authority.is_zero() == false, Error::<T>::T1AuthorityIsMandatory);
    ensure!(total_supply > 0u64, Error::<T>::TotalSupplyZero);

    Self::validate_royalties(&royalties)?;

    let sender_nonce = Self::batch_nonce(&sender);
    let signed_payload = encode_create_batch_params::<T>(&proof, &royalties, &t1_authority, &total_supply, &sender_nonce);
    ensure!(
        Self::verify_signature(&proof, &signed_payload.as_slice()).is_ok(),
        Error::<T>::UnauthorizedSignedCreateBatchTransaction
    );

    let batch_id = generate_batch_id::<T>(Self::get_unique_id_and_advance());
    ensure!(BatchInfoId::contains_key(&batch_id) == false, Error::<T>::BatchAlreadyExists);

    // No errors allowed after this point because `get_info_id_and_advance` mutates storage
    let info_id = Self::get_info_id_and_advance();
    create_batch::<T>(info_id, batch_id, royalties, total_supply, t1_authority, sender.clone());

    <BatchNonces<T>>::mutate(&sender, |n| *n += 1);

    Self::deposit_event(RawEvent::BatchCreated(batch_id, total_supply, sender, t1_authority));

    Ok(())
}
```

Creating a new batch with a total supply of 50 doesn't automatically mint all 50 instances of that NFT. NFTs are minted when needed, for example, at the point of sale. The minting process has a validation step which checks that the batch is listed for sale and that there isn't an attempt to mint more NFTs to a batch than what was stated as the `total_supply`.

```rust
pub fn validate_mint_batch_nft_request<T: Config>(
    batch_id: NftBatchId,
    unique_external_ref: &Vec<u8>) -> Result<NftInfo<T::AccountId>, DispatchError>
{
    ensure!(batch_id.is_zero() == false, Error::<T>::BatchIdIsMandatory);
    ensure!(BatchInfoId::contains_key(&batch_id), Error::<T>::BatchDoesNotExist);
    ensure!(<BatchOpenForSale>::contains_key(&batch_id) == true, Error::<T>::BatchNotListed);

    let nft_info = get_nft_info_for_batch::<T>(&batch_id)?;
    ensure!((NftBatches::get(&batch_id).len() as u64) < nft_info.total_supply, Error::<T>::TotalSupplyExceeded);

    Module::<T>::validate_external_ref(unique_external_ref)?;

    Ok(nft_info)
}
```

```rust
/// Mints an nft that belongs to a batch
#[weight = T::WeightInfo::proxy_signed_mint_batch_nft()]
fn signed_mint_batch_nft(origin,
    proof: Proof<T::Signature, T::AccountId>,
    batch_id: NftBatchId,
    index: u64,
    owner: T::AccountId,
    unique_external_ref: Vec<u8>) -> DispatchResult
{
    let sender = ensure_signed(origin)?;
    ensure!(sender == proof.signer, Error::<T>::SenderIsNotSigner);

    let nft_info = validate_mint_batch_nft_request::<T>(batch_id, &unique_external_ref)?;
    ensure!(<BatchOpenForSale>::get(&batch_id) == NftSaleType::Fiat, Error::<T>::BatchNotListedForFiatSale);
    ensure!(nft_info.creator == Some(sender), Error::<T>::SenderIsNotBatchCreator);

    let signed_payload = encode_mint_batch_nft_params::<T>(&proof, &batch_id, &index, &unique_external_ref, &owner);
    ensure!(
        Self::verify_signature(&proof, &signed_payload.as_slice()).is_ok(),
        Error::<T>::UnauthorizedSignedMintBatchNftTransaction
    );

    mint_batch_nft::<T>(batch_id, owner, index, &unique_external_ref)?;

    Ok(())
}
```

Batch nft sale is a two step process.

- Create the batch which will generate the ID.
- Sell nft and mint on the AvN.

```rust
pub fn encode_list_batch_for_sale_params<T: Config>(
    proof: &Proof<T::Signature, T::AccountId>,
    batch_id: &NftBatchId,
    market: &NftSaleType,
    nonce: &u64) -> Vec<u8>
{
    return (
        SIGNED_LIST_BATCH_FOR_SALE_CONTEXT,
        &proof.relayer,
        batch_id,
        market,
        nonce
    ).encode();
}
```

List a batch NFT for sale checks that the transaction originator is the `creator` of the batch and the batch does exist. The batch can be relisted by the creator as long as there are free (unsold) nfts in the batch.

```rust
#[weight = T::WeightInfo::proxy_signed_list_batch_for_sale()]
fn signed_list_batch_for_sale(origin,
    proof: Proof<T::Signature, T::AccountId>,
    batch_id: NftBatchId,
    market: NftSaleType,
) -> DispatchResult
{
    let sender = ensure_signed(origin)?;
    ensure!(sender == proof.signer, Error::<T>::SenderIsNotSigner);
    ensure!(batch_id.is_zero() == false, Error::<T>::BatchIdIsMandatory);
    ensure!(BatchInfoId::contains_key(&batch_id), Error::<T>::BatchDoesNotExist);
    ensure!(market != NftSaleType::Unknown, Error::<T>::UnsupportedMarket);

    let sender_nonce = Self::batch_nonce(&sender);
    let nft_info = get_nft_info_for_batch::<T>(&batch_id)?;
    //Only the batch creator can allow mint operations.
    ensure!(nft_info.creator == Some(sender.clone()), Error::<T>::SenderIsNotBatchCreator);
    ensure!((NftBatches::get(&batch_id).len() as u64) < nft_info.total_supply, Error::<T>::NoNftsToSell);
    ensure!(<BatchOpenForSale>::contains_key(&batch_id) == false, Error::<T>::BatchAlreadyListed);

    let signed_payload = encode_list_batch_for_sale_params::<T>(&proof, &batch_id, &market, &sender_nonce);
    ensure!(
        Self::verify_signature(&proof, &signed_payload.as_slice()).is_ok(),
        Error::<T>::UnauthorizedSignedListBatchForSaleTransaction
    );

    <BatchOpenForSale>::insert(batch_id, market);
    <BatchNonces<T>>::mutate(&sender, |n| *n += 1);

    Self::deposit_event(RawEvent::BatchOpenForSale(batch_id, market));

    Ok(())
}
```

Finally, only the creator of a batch can end the open sale listing of a batch.

```rust
pub fn encode_end_batch_sale_params<T: Config>(
    proof: &Proof<T::Signature, T::AccountId>,
    batch_id: &NftBatchId,
    nonce: &u64) -> Vec<u8>
{
    return (
        SIGNED_END_BATCH_SALE_CONTEXT,
        &proof.relayer,
        batch_id,
        nonce
    ).encode();
}
```

```rust
#[weight = T::WeightInfo::proxy_signed_end_batch_sale()]
fn signed_end_batch_sale(origin,
    proof: Proof<T::Signature, T::AccountId>,
    batch_id: NftBatchId,
) -> DispatchResult {
    let sender = ensure_signed(origin)?;
    ensure!(sender == proof.signer, Error::<T>::SenderIsNotSigner);
    validate_end_batch_listing_request::<T>(&batch_id)?;
    ensure!(<BatchOpenForSale>::get(&batch_id) == NftSaleType::Fiat, Error::<T>::BatchNotListedForFiatSale);

    let sender_nonce = Self::batch_nonce(&sender);
    let nft_info = get_nft_info_for_batch::<T>(&batch_id)?;
    //Only the batch creator can end the listing
    ensure!(nft_info.creator == Some(sender.clone()), Error::<T>::SenderIsNotBatchCreator);

    let signed_payload = encode_end_batch_sale_params::<T>(&proof, &batch_id, &sender_nonce);
    ensure!(
        Self::verify_signature(&proof, &signed_payload.as_slice()).is_ok(),
        Error::<T>::UnauthorizedSignedEndBatchSaleTransaction
    );

    let market = <BatchOpenForSale>::get(batch_id);
    end_batch_listing::<T>(&batch_id, market)?;
    <BatchNonces<T>>::mutate(&sender, |n| *n += 1);

    Ok(())
}
```

New endpoints created for managing batch NFTs.

```rust
Call::NftManager(pallet_nft_manager::Call::signed_create_batch(proof, _, _, _)) => return Some(proof.clone()),
Call::NftManager(pallet_nft_manager::Call::signed_mint_batch_nft(proof, _, _, _, _)) => return Some(proof.clone()),
Call::NftManager(pallet_nft_manager::Call::signed_list_batch_for_sale(proof, _, _)) => return Some(proof.clone()),
Call::NftManager(pallet_nft_manager::Call::signed_end_batch_sale(proof, _)) => return Some(proof.clone()),
```

## Backwards Compatibility

This standard is backward compatible with [AIP-1](./AIP-1).

## Tests

The batch NFT tests can be found [HERE](../../assets/AIP-2/batch_nft_tests.rs).

## Reference Implementation

This proposal is dependent on the following AIPs below:

- <a href="./AIP-1">AIP-1</a>.

## Copyright

Copyright and related rights waived via [CC0](../../../LICENSE)

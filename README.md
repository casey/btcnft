# BTC NFT

BTC NFT is a bitcoin-native protocol for minting and transferring NFTs.

BTC NFTs are associated with individual satoshis, and are transferred using
simple bitcoin transactions.

## Satpoints

The current location of a BTC NFT is a satpoint. A satpoint consists of a
transaction ID, an output index, and a satoshi offset, separated by colons.

This satpoint:

```
4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b:0:495716253
```

Has the following parts:

- Transaction ID:
  `4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b`
- Output index: `0`
- Satoshi offset: `495716253`

A transaction ID and output index, known as an outpoint, identify a particular
output. A satpoint identifies a particular satoshi within an outpoint.

## Minting BTC NFTs

BTC NFTs are minted by signing a message containing the NFT metadata, which
includes a satpoint. The metadata for an image NFT, represented as JSON, might
look like this:

```json
{
  "image": "b29933e0cc8490ffd1ba687a684c78fa0d5dd0c1089e39f03b4c4be068abfabc",
  "genesis": "4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b:0:495716253",
}
```

Whose fields have the following meaning:

- `image`: The SHA-256 hash of the NFTs image data.
- `genesis`: The initial satpoint to which the NFT is assigned.

## NFT IDs

The SHA-256 hash of the signature of an NFT serves as a compact identifier of
a given NFT. This NFT ID can be used to look up the signature and metadata of
an NFT.

Looking up the metadata of an NFT ID is trustless, since NFT ID commits to the
signature, and the signature commits to the metadata, so NFT metadata can be
stored anywhere, for example in distributed P2P networks such as BitTorrent or
IPFS, or on centralized NFT metadata repositories.

## NFT Storage

NFT metadata can be trustlessly retrieved using an NFT ID, so they can be
stored anywhere.

## Transferring BTC NFTs

BTC NFTs are transferred when the satpoint holding the NFT is spent, using the
following algorithm:

```python
def transfer(satpoint, transaction):
  offset = 0

  for input in transaction.inputs:
    if input == satpoint.output:
      offset += satpoint.offset
      break
    else:
      offset += input.value

  for output in transaction.outputs:
    if output.value > offset:
      return Satpoint(output, offset)
    else:
      offset -= output.value

  return None
```

This algorithm first calculates the offset, in satoshis, of the satpoint within
the inputs of the transaction. It then calculates where that offset places the
satpoint within the outputs of the transaction, and returns a new satpoint
pointing to the new offset.

If the new offset points to the fee range, the `transfer` function returns
`None`. For simplicity, the meaning of such a transfer is left undefined.
Depending on future needs, such a transfer may be defined as destroying any
associated NFTs, or be defined as assigning any associated NFTs to the miner of
the current block.

## Finding the current owner of a BTC NFT

The current owner of a BTC NFT given by a particular NFT ID can be found by
retrieving the NFT metadata from a metadata repository using the NFT ID. Then,
starting from the genesis satpoint, the transfer algorithm is applied whenever
the current satpoint is spent.

In code:

```
def find(nft):
  current = nft.genesis

  while spent(current):
    spending_transaction = get_spending_transaction(current.outpoint):

    current = transfer(current, spending_transaction)

  return current
```

The owner of the output in which the NFT currently resides is the owner of the
NFT, and the public key that controls that output maybe be used to sign
ownership challenges related to that NFT.

## Relationship to Ordinals

[Ordinals](https://ordinals.com) is a scheme for collectable satoshis, and
associated NFTs, and uses a similar transfer scheme. Ordinals is more fun and
aesthetically appealing than BTC NFT. However, proofs of an ordinal NFTs
location are larger, and building a database over the location of all ordinals
is resource intensive.

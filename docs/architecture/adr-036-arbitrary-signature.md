# ADR 036: Arbitrary Message Signature Specification

## Changelog

- 28/10/2020 - Initial draft

## Authors
- Antoine Herzog (@antoineherzog)
- Zaki Manian (@zmanian)
- Aleksandr Bezobchuk (alexanderbez) [1]
- Frojdi Dymylja (@fdymylja)

## Status

Draft

## Abstract

Currently, in the SDK, there is no convention to sign arbitrary message like on Ethereum. We propose with this specification, for Cosmos SDK ecosystem, a way to sign and validate off-chain arbitrary messages.

This specification serves the purpose of covering every use case, this means that cosmos-sdk applications developers decide how to serialize and represent `Data` to users. 
## Context

Having the ability to sign messages off-chain has proven to be a fundamental aspect of nearly any blockchain. The notion of signing messages off-chain has many added benefits such as saving on computational costs and reducing transaction throughput and overhead. Within the context of the Cosmos, some of the major applications of signing such data includes, but is not limited to, providing a cryptographic secure and verifiable means of proving validator identity and possibly associating it with some other framework or organization. In addition, having the ability to sign Cosmos messages with a Ledger or similar HSM device.


## Decision

The aim is being able to sign arbitrary messages, even using Ledger or similar HSM devices.

As a result signed messages should look roughly like Cosmos SDK messages but **must not** be a valid on-chain transaction. `chain-id`, `account_number` and `sequence` can all be assigned invalid values.

Cosmos SDK 0.40 also introduces a concept of “auth_info” this can specify SIGN_MODES.

A spec should include an `auth_info` that supports SIGN_MODE_DIRECT and SIGN_MODE_LEGACY_AMINO.

- the memo must be empty
- nonce, sequence number must be equal to 0
- chain-id must be equal to “signature”
- fee gas must be equal to 0
- fee amount must be an empty array
- inside the message with type `MsgSignData`, we put inside *bytes* data and the `address` of the signer.
- it's applications developers decision how `Data` should be treated, by treated we mean the serialization and deserialization process and the Object `Data` should represent. 

Proto definition:
```proto
// MsgSignData defines an arbitrary, general-purpose, off-chain message
message MsgSignData {
    // Signer is the sdk.AccAddress of the message signer
    bytes Signer = 1 [(gogoproto.jsontag) = "signer", (gogoproto.casttype) = "github.com/cosmos/cosmos-sdk/types.AccAddress"];
    // Data represents the raw bytes of the content that is signed (text, json, etc)
    bytes Data = 2 [(gogoproto.jsontag) = "data"];
}
```
Signed MsgSignData json example:
```json
{
  "type": "cosmos-sdk/StdTx",
  "value": {
    "msg": [
      {
        "type": "sign/MsgSignData",
        "value": {
          "signer": "cosmos1hftz5ugqmpg9243xeegsqqav62f8hnywsjr4xr",
          "data": "cmFuZG9t"
        }
      }
    ],
    "fee": {
      "amount": [],
      "gas": "0"
    },
    "signatures": [
      {
        "pub_key": {
          "type": "tendermint/PubKeySecp256k1",
          "value": "AqnDSiRoFmTPfq97xxEb2VkQ/Hm28cPsqsZm9jEVsYK9"
        },
        "signature": "8y8i34qJakkjse9pOD2De+dnlc4KvFgh0wQpes4eydN66D9kv7cmCEouRrkka9tlW9cAkIL52ErB+6ye7X5aEg=="
      }
    ],
    "memo": ""
  }
}
```

## Consequences

There is a specification on how messages, that are not meant to be broadcast to a live chain, should be formed. 

### Backwards Compatibility

Backwards compatibility is maintained as this is a new message spec definition.

### Positive

- A common format that can be used by multiple applications to sign and verify off-chain messages.
- The specification is primitive which means it can cover every use case without limiting what is possible to fit inside it.
- It gives room for other off-chain messages specifications that aim to target more specific and common use cases such as off-chain-based authN/authZ layers [2].

### Negative



## References

1. https://github.com/cosmos/ics/pull/33
2. https://github.com/cosmos/cosmos-sdk/pull/7727#discussion_r515668204
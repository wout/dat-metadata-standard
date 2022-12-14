# Venster Metadata Standard (Draft)
A metadata standard for storing distributed on-chain NFTs on Cardano.

**IMPORTANT**: This document is still a work in progress. 

## Introduction
This standard describes the separation of data and logic for on-chain NFTs. It
is intended for generative on-chain art but it may also apply to other use
cases. The main goal is to solve the three problems described below.

### **Problem 1**: Storage limit

Cardano is very well suited for on-chain NFTs. Compared to other blockchains,
Cardano has the lowest L1 storage cost per kB, but the maximum transaction size
of 16 kB is more limited in comparison to other chains.

### **Problem 2**: Inefficient use of storage

Some existing on-chain projects on Cardano make inefficient use of block space
by repeatedly storing the same monolithic blob accompanied by a few unique
parameters. This results in thousands of copies of the same code, often close to
or at the full capacity of the 16 kB limit.

### **Problem 3**: External dependencies

Sometimes it may not be feasible, or even impossible, to store all dependencies
on the blockchain. Examples are p5.js, three.js, python or Blender to name a
few. There is no clearly defined way to describe external dependencies in such a
way that on-chain NFTs can be reproduced by third parties.

## Metadata

The Venster Metadata Standard builds on the existing
[CIP-0025](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0025)
standard and is divided into three separate entities.

### **1**. Scene

The *scene* token is the part the end user will receive in their wallet. It
contains all the information to render the NFT.

```
{
  "721": {
    "<policy_id>": {
      "<asset_name>": {
        "name": <string>,
        "description": <string>,

        "image": <uri | array>,
        "mediaType": image/<mime_sub_type>,
        
        "files": [{
          "name": <string>,
          "mediaType": <mime_type>,
          "src": <uri | array>,
          <other_properties>
        }],

        "renderer": {
          "main": <renderer_asset_name>,
          "arguments": <array>
        }
      }
    }
  }
}
```

Properties for the *scene* token:
- **`renderer`** (required): an object with two properties
  - **`main`** (required): the `asset_name` of the renderer token within the
    current `policy_id` (e.g. `my_renderer`)
  - **`arguments`** (required): an array with arbitrary values used as
    arguments for the invocation of the renderer (e.g. `[123]`)

#### Dynamic arguments

Several dynamic arguments can be passed to the renderer:
- `@txhash` (`string`): transaction hash of the mint (can be used as the seed
  value for an Sfc32 PRNG for example)
- `@txhashes` (`string[]`): an array of all transaction hashes of the token
- `@epoch` (`number`): epoch in which the token was minted
- `@slot` (`number`): slot in which the token was minted
- `@block` (`number`): block in which the token was minted
- `@block_size` (`number`): size of the block
- `@block_output` (`string`): total output (Lovelace) of the block
- `@current_epoch` (`number`): current (latest) epoch
- `@current_slot` (`number`): current (latest) slot
- `@current_block` (`number`): current (latest) minted block
- `@current_block_size` (`number`): size of the current block
- `@current_block_output` (`string`): total output (Lovelace) of the current
  block

Dynamic arguments can be defined just like regular arguments:

```
[
  123,
  "@txhash",
  "@block",
  "@current_block"
]
```

### **2**. Renderer

The *renderer* token is part of the same `policy_id`. It can either be a
self-contained on-chain program or one with dependencies. Within the same
policy, multiple *renderer* tokens can exist, but *scene* tokens can only
reference one at a time.

The code is stored in the **`files`** property as-is or as a base64-encoded
string. The `name` property of the file should match the `asset_name`.

```
{
  "721": {
    "<policy_id>": {
      "<asset_name>": {
        "files": [{
          "name": <asset_name>.<extension>,
          "mediaType": <mime_type>,
          "src": <uri | array>,
          <other_properties>
        }],

        "outputType": <mime_type>,

        "dependencies": [{
          "type": <string>,
          <other_properties>
        }],

        "license": <string | null>
      }
    }
  }
}
```

Properties for the *renderer* token:
- **`outputType`** (required): the mime type of the renderer's output (it's up
  to the viewer to define de accepted formats)
- **`dependencies`** (optional): an array of objects with dependency
  definitions
- **`license`** (optional): name and url of the copyright license for the
  renderer (e.g. `NFT License 2.0`; more info below)

**Note**: The renderer token can be burned after minting to free up the UTxO.

#### On-chain dependencies

These are project-specific dependencies managed by the minter. They should be
minted within the same `policy_id`.

```
{
  "type": "onchain",
  "asset_name": <dependency_asset_name>
}
```

#### Internal dependencies:

These are on-chain dependencies managed by the viewer and made available to the
*renderer* on execution.

```
{
  "type": "internal",
  "fingerprint": <asset_fingerprint>
}
```

#### External dependencies:

These are off-chain dependencies managed by the viewer and made available to the
*renderer* on execution.

```
{
  "type": "external",
  "name": <library_name>,
  "version": <version_number>
}
```

### License types

It is recommended to choose a license that aligns with the values of the
creator. Popular licenses are:

- [NFT License 2.0](https://www.nftlicense.org/)
- [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
- [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- [AGPL 3.0](https://www.gnu.org/licenses/agpl-3.0.en.html)


Using [no license](https://choosealicense.com/no-permission/) is also an option
to indicate that the software may not be used by anyone else than the creator.

### **3**. Dependency

A *dependency* token is part of the same `policy_id`. Its code is stored in the
**`files`** property as-is or as a base64-encoded string. The `name` property of
the file should match the `asset_name`.

```
{
  "721": {
    "<policy_id>": {
      "<asset_name>": {
        "files": [{
          "name": <asset_name>.<extension>,
          "mediaType": <mime_type>,
          "src": <uri | array>,
          "license": <string | null>,
          <other_properties>
        }]
      }
    }
  }
}
```

**Note**: Dependency tokens can be burned after minting to free up the UTxOs.

## License
Venster Metadata Standard is licensed under
[CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/legalcode).

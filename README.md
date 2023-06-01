# DAT Metadata Standard

A metadata standard for storing Distributed Artifact Tokens, or DATs, on the Cardano blockchain.

## Introduction

This standard is intended for on-chain generative art but may also be suitable for other use cases. In essence, it describes a method to store generative tokens on the blockchain in a distributed and space-efficient way using Cardano's Native Tokens. Its main goal is to solve the four problems described below.

### **Problem 1**: Storage limit

Cardano is very well suited for on-chain generative art. Compared to other blockchains, it has the lowest L1 storage cost per kB, but the maximum transaction size of 16 kB is more limited in comparison to other chains.

### **Problem 2**: Inefficient use of block space

Some existing on-chain projects on Cardano make inefficient use of block space by repeatedly storing the same monolithic blob accompanied by a few unique parameters. The result is thousands of copies of the same code, close to or at the full capacity of the 16 kB limit.

### **Problem 3**: External dependencies

Storing all dependencies for a generative token on the blockchain isn't always convenient or even viable. Examples are p5.js, three.js, python or Blender, to name a few. There is no clearly defined way to describe external dependencies so that digital artifacts can be easily reproduced by third parties.

### **Problem 4**: Token properties

As it is right now, a token's root namespace is often polluted with arbitrary key/value pairs. It makes sense to constrain token-specific properties to a `properties` object, making it easier for viewers to locate and render those values.

## DATs

Generative tokens creating following this standard are called **Distributed Artifact Tokens**, or **DAT**s. They can be fungible, semi-fungible or non-fungible. They are not necessarily a replacement for NFTs, but rather a separate class of token in their own right.

Aside from their distributed nature, DATs can query information about the current state of the blockchain, from their own mint transaction, and from previously minted tokens to create inter-linked token collections. In that sense, DATs facilitate artists to create art _with_ the blockchain.

## Metadata

The DAT Metadata Standard builds on the existing [CIP-0025](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0025) standard and is divided into three separate entities.

### **1**. Scene

The *scene* token is the part the end user will receive in their wallet. It contains all the information to render the DAT. The scene token requires a `renderer` property to be present, referencing the renderer token.

```cddl
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
          "license": <string>,
          <other_properties>
        }],

        "properties": {
          <properties>
        },

        "renderer": {
          "main": <asset_name>,
          "arguments": <array>
        }
      }
    }
  }
}
```

Properties for the *scene* token:
- **`renderer`** (_required_): an object with two properties
  - **`main`** (_required_): the `asset_name` of the renderer token within the
    current `policy_id` (e.g. `my_renderer`)
  - **`arguments`** (_required_): an array with arbitrary values used as
    arguments for the invocation of the renderer (e.g. `[123]`)
- **`properties`** (_optional_): an object with arbitrary key/value pairs describing the token's (unique) properties


**Note**: If a DAT uses data from the blockchain to drive its generative algorithm, it's impossible to generate a thumbnail image before the mint. So the preview image for wallets and marketplaces should be treated more as an album cover than a visual representation of the actual artifact.

#### **1.a.** Argument directives

Several directives for dynamic arguments can be passed to the renderer:

_Current token_
- `@tx_hash` (`string`): transaction hash of the mint (can for example be used as the seed value for a Sfc32 PRNG)
- `@epoch` (`number`): epoch in which the token was minted
- `@slot` (`number`): slot in which the token was minted
- `@block` (`number`): block in which the token was minted
- `@block_size` (`number`): size of the token's block
- `@block_hash` (`string`): hash of the token's block

_Previously minted token_
- `@tx_hash.previous` (`string | null`): transaction hash
- `@epoch.previous` (`number | null`): epoch in which the token was minted
- `@slot.previous` (`number | null`): slot in which the token was minted
- `@block.previous` (`number | null`): block in which the token was minted
- `@block_size.previous` (`number | null`): size of the token's block
- `@block_hash.previous` (`string | null`): hash of the token's block
- `@arguments.previous` (`array | null`): token's renderer arguments

_Specific token (within the same policy_id)_
- `@tx_hash.asset_name` (`string | null`): transaction hash
- `@epoch.asset_name` (`number | null`): epoch in which the token was minted
- `@slot.asset_name` (`number | null`): slot in which the token was minted
- `@block.asset_name` (`number | null`): block in which the token was minted
- `@block_size.asset_name` (`number | null`): size of the token's block
- `@block_hash.asset_name` (`string | null`): hash of the token's block
- `@arguments.asset_name` (`array | null`): token's renderer arguments

_Current blockchain state_
- `@current_epoch` (`number`): current (latest) epoch
- `@current_slot` (`number`): current (latest) slot
- `@current_block` (`number`): current (latest) minted block
- `@current_block_size` (`number`): size of the current block
- `@current_block_hash` (`string`): hash of the current block

Arguments directives can be passed to the renderer just like regular arguments:

```json
[
  123,
  "@tx_hash",
  "@block",
  "@current_block",
  "@epoch.previous",
  "@arguments.the_perfect_nft_000"
]
```

**Note**: Referencing another token's arguments does not work recursively.

### **2**. Renderer

The *renderer* token is part of the same `policy_id`. It can either be a self-contained program or one with dependencies. Within the same policy, multiple *renderer* tokens can exist, but *scene* tokens can only reference one at a time.

The code is stored in the `files` property as-is or as a base64-encoded string. The `name` property of the file should match the `asset_name`.

Instructions and/or requirements to reproduce the token can be stored in the `instructions` property. For browser-based artworks, this should include the current browser version(s) in which it works. For projects executed locally, it should be a dependency file for a package manager (examples below).

```cddl
{
  "721": {
    "<policy_id>": {
      "<asset_name>": {
        "files": [{
          "name": <asset_name>.<extension>,
          "mediaType": <mime_type>,
          "src": <uri | array>,
          "license": <string>,
          <other_properties>
        }],

        "outputType": <mime_type>,

        "dependencies": [{
          "type": <string>,
          <other_properties>
        }],

        "instructions": <string | array>
      }
    }
  }
}
```

Properties for the *renderer* token:
- **`outputType`** (_required_): the mime type of the renderer's output (it's up to the viewer to define the supported formats)
- **`dependencies`** (_optional_): an array of objects with dependency definitions
- **`instructions`** (_optional_): a text string or an array of text strings

While not mandatory, it's advisable to add a **`license`** property to each file in the `files` section. More info on licenses below in [section 2.d.](https://github.com/venster-io/venster-metadata-standard#2d-license-types)

**Note**: The renderer token should be burned after minting to free up the UTxO.

#### **2.a.** On-chain dependencies

These are project-specific dependencies managed by the minter. They should be minted within the same `policy_id`.

```cddl
{
  "type": "onchain",
  "asset_name": <asset_name>
}
```

#### **2.b.** Internal dependencies:

These are on-chain dependencies managed by the viewer and made available to the *renderer* on execution.

```cddl
{
  "type": "internal",
  "fingerprint": <asset_fingerprint>
}
```

Or

```cddl
{
  "type": "internal",
  "policy_id": <policy_id>,
  "asset_name": <asset_name>
}
```

#### **2.c.** External dependencies:

These are off-chain dependencies managed by the viewer and made available to the *renderer* on execution.

```cddl
{
  "type": "external",
  "name": <library_name>,
  "version": <version_number>
}
```

#### **2.d.** License types

It is recommended to choose a license that aligns with the values of the creator and purpose of the digital artifact. Popular licenses are:

- [NFT License 2.0](https://www.nftlicense.org/)
- [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
- [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- [AGPL 3.0](https://www.gnu.org/licenses/agpl-3.0.en.html)

Using [no license](https://choosealicense.com/no-permission/) is also an option to indicate that no one other than the creator may use the software.

### **3**. Dependency

A *dependency* token is part of the same `policy_id`. Its code is stored in the **`files`** property as-is or as a base64-encoded string. The `name` property of the file should match the `asset_name`. Similar to the *renderer*, every file can have an individual `license` property.

Dependencies can consist of multiple parts if they don't fit into one 16kB transaction. The dependency referenced from the renderer serves as an entrypoint referencing the additional parts, and should contains the first part as well. Viewers can decide how many parts to allow or support.

```cddl
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
        }],

        "parts": [
          <part_asset_name>
        ]
      }
    }
  }
}
```

Properties for the *dependency* token:
- **`parts`** (_optional_): an array with asset names (e.g. `asset_name_part_2`)

While not mandatory, it's advisable to add a **`license`** property to each file in the `files` section. More info on licenses in [section 2.d.](https://github.com/venster-io/venster-metadata-standard#2d-license-types)

**Note**: Dependency tokens should be burned after minting to free up the UTxOs.

## License
DAT Metadata Standard is licensed under
[CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/legalcode).

# Venster Metadata Standard (Beta)
A metadata standard for storing modular on-chain NFTs on Cardano.

## Introduction
This standard describes the separation of data and logic for on-chain NFTs. It
is intended for generative on-chain art but it may also be suitable for other
use cases. The main goal is to solve the four problems described below.

### **Problem 1**: Storage limit

Cardano is very well suited for on-chain NFTs. Compared to other blockchains,
Cardano has the lowest L1 storage cost per kB, but the maximum transaction size
of 16 kB is more limited in comparison to other chains.

### **Problem 2**: Inefficient use of block space

Some existing on-chain projects on Cardano make inefficient use of block space by repeatedly storing the same monolithic blob accompanied by a few unique parameters. This results in thousands of copies of the same code, often close to or at the full capacity of the 16 kB limit.

### **Problem 3**: External dependencies

Storing all dependencies for a generative artwork on the blockchain isn't always an option. Examples are p5.js, three.js, python or Blender, to name a few. There is no clearly defined way to describe external dependencies in such a way that on-chain NFTs can be reproduced by third parties.

### **Problem 4**: NFT properties

An asset's root namespace is often polluted with arbitrary key/value pairs. It makes sense to constrain token-specific properties to a `properties` object, making it easier for viewers to locate and render those values.

## Metadata

The Venster Metadata Standard builds on the existing [CIP-0025](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0025) standard and is divided into three separate entities.

### **1**. Scene

The *scene* token is the part the end user will receive in their wallet. It contains all the information to render the NFT. This part adds a `renderer` property to the CIP 25 standard:

```
{
  "721": {
    "<policy_id>": {
      "<asset_name>": {
        "name": <string>,
        "description": <string>,

        "image": <uri | array>,
        "mediaType": image/<mime_sub_type>,
        "blurhash": <string>,
        
        "files": [{
          "name": <string>,
          "mediaType": <mime_type>,
          "src": <uri | array>,
          <other_properties>
        }],

        "properties": {
          <properties>
        },

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
- **`properties`** (optional): an object with arbitrary key/value pairs describing the token's (unique) properties
- **`blurhash`** (optional): a thumb image placeholder using the [blurhash algorithm](https://github.com/woltapp/blurhash)

**Note**: A `blurhash` can be used instead of the `image` property to remove the external storage dependency (IPFS, Arweave, ...). That way, the token is fully on-chain. Wallets or viewers can use it as a stand-in thumb image without the overhead or rendering the complete token. Blurhash strings can be rendered on a canvas element or converted to a base64 png data string by clients.

#### Directives

Several directives for dynamic arguments can be passed to the renderer:
- `@tx_hash` (`string`): transaction hash of the mint (can be used as the seed value for an Sfc32 PRNG for example)
- `@epoch` (`number`): epoch in which the token was minted
- `@slot` (`number`): slot in which the token was minted
- `@block` (`number`): block in which the token was minted
- `@block_size` (`number`): size of the token's block
- `@block_hash` (`string`): hash of the token's block
- `@properties` (`object`): token's properties object
- `@previous_tx_hash` (`string`): transaction hash of the previous mint
- `@previous_epoch` (`number`): epoch in which the previous token was minted
- `@previous_slot` (`number`): slot in which the previous token was minted
- `@previous_block` (`number`): block in which the previous token was minted
- `@previous_block_size` (`number`): size of the previous token's block
- `@previous_block_hash` (`string`): hash of the previous token's block
- `@previous_properties` (`object`): previous token's properties object
- `@current_epoch` (`number`): current (latest) epoch
- `@current_slot` (`number`): current (latest) slot
- `@current_block` (`number`): current (latest) minted block
- `@current_block_size` (`number`): size of the current block
- `@current_block_hash` (`string`): hash of the current block

Directives can be defined just like regular arguments:

```
[
  123,
  "@tx_hash",
  "@block",
  "@current_block"
]
```

### **2**. Renderer

The *renderer* token is part of the same `policy_id`. It can either be a self-contained on-chain program or one with dependencies. Within the same policy, multiple *renderer* tokens can exist, but *scene* tokens can only reference one at a time.

The code is stored in the **`files`** property as-is or as a base64-encoded string. The `name` property of the file should match the `asset_name`.

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
        }],

        "outputType": <mime_type>,

        "dependencies": [{
          "type": <string>,
          <other_properties>
        }]
      }
    }
  }
}
```

Properties for the *renderer* token:
- **`outputType`** (required): the mime type of the renderer's output (it's up
  to the viewer to define the supported formats)
- **`dependencies`** (optional): an array of objects with dependency
  definitions

Please consider adding a **`license`** property to the renderer file(s). More info on licenses below.

**Note**: The renderer token can be burned after minting to free up the UTxO.

#### On-chain dependencies

These are project-specific dependencies managed by the minter. They should be minted within the same `policy_id`.

```
{
  "type": "onchain",
  "asset_name": <dependency_asset_name>
}
```

#### Internal dependencies:

These are on-chain dependencies managed by the viewer and made available to the *renderer* on execution.

```
{
  "type": "internal",
  "fingerprint": <asset_fingerprint>
}
```

#### External dependencies:

These are off-chain dependencies managed by the viewer and made available to the *renderer* on execution.

```
{
  "type": "external",
  "name": <library_name>,
  "version": <version_number>
}
```

### License types

It is recommended to choose a license that aligns with the values of the creator. Popular licenses are:

- [NFT License 2.0](https://www.nftlicense.org/)
- [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
- [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- [AGPL 3.0](https://www.gnu.org/licenses/agpl-3.0.en.html)

Using [no license](https://choosealicense.com/no-permission/) is also an option to indicate that no one other than the creator may use the software.

### **3**. Dependency

A *dependency* token is part of the same `policy_id`. Its code is stored in the **`files`** property as-is or as a base64-encoded string. The `name` property of the file should match the `asset_name`. Similar to the *renderer*, every file can have an individual `license` property.

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

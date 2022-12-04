# Venster Metadata Standard (Draft)
A metadata standard for storing distributed on-chain NFTs on Cardano.

**IMPORTANT**: This document is still a work in progress. 

## Introduction
This standard describes the separation of data and logic for on-chain NFTs. It
is intended for generative on-chain art but it may also apply to other use
cases. The main goal is to solve the four problems described below.

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

### **Problem 4**: Inconsistent project properties
Properties of existing projects are often messy and unstructured, with odd and
inconsistent key names.

## Metadata
The Venster Metadata Standard builds on the existing
[CIP-0025](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0025)
standard and is divided into three separate entities.

### **1**. Scene
This is the part the end user will receive in their wallet. It contains all the
information to render the NFT.

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
          "arguments": <object>
        },

        "properties": <array | null>
      }
    }
  }
}
```

Properties for the **scene** entity:
- **`renderer`** (required): an object with two properties
  - **`main`** (required): the `asset_name` of the renderer token within the
    current `policy_id` (e.g. `my_renderer`)
  - **`arguments`** (required): an object with arbitrary key/value pairs used as
    arguments for the invocation of the renderer (e.g. `{"seed": 123}`)
- **`properties`** (optional): an object with arbitrary key/value pairs
  describing the project

### **2**. Renderer
The renderer token is part of the same `policy_id`. It can either be a
self-contained program or one with external dependencies. The code is stored in
the **`files`** property as-is or as a base64-encoded string. The `name`
property of the file should be the same as the `asset_name`.

```
{
  "721": {
    "<policy_id>": {
      "<asset_name>": {
        "files": [{
          "name": <asset_name>,
          "mediaType": <mime_type>,
          "src": <uri | array>
        }],

        "outputType": <mime_type>,

        "dependencies": <array | null>
      }
    }
  }
}
```

Properties for the **renderer** entity:
- **`outputType`** (required): the mime type of the renderer's output
- **`dependencies`** (optional): an array of objects with dependency
  definitions.

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
renderer on execution.

```
{
  "type": "internal",
  "fingerprint": <asset_fingerprint>
}
```

#### External dependencies:

These are managed by the viewer and made available to the renderer on execution.

```
{
  "type": "external",
  "name": <library_name>,
  "version": <version_number>
}
```

### **3**. Dependency
A dependency token is part of the same `policy_id`. Its code is stored in the
**`files`** property as-is or as a base64-encoded string. The `name` property of
the file should be the same as the `asset_name`.

```
{
  "721": {
    "<policy_id>": {
      "<asset_name>": {
        "files": [{
          "name": <asset_name>,
          "mediaType": <mime_type>,
          "src": <uri | array>
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

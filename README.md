# Elpis Metadata Standard (v1.0)
A metadata standard for storing distributed on-chain NFTs on Cardano.

## Introduction
This standard describes the separation of data and logic for on-chain NFTs. It
is useful for generative on-chain art but it may also apply to other use cases.
The main goal is to solve the three problems as described below.

### Storage limit
Cardano is very well suited for on-chain NFTs. Compared to other blockchains,
Cardano has the lowest L1 storage cost per kB, but the maximum transaction size
of 16 kB is more limited in comparison to other chains.

### Inefficient use of storage
Some existing on-chain projects on Cardano make inefficient use of block space
by repeatedly storing the same monolithic blob accompanied by a few unique
parameters. This results in thousands of copies of the same code, often close to
or at the full capacity of the 16 kB limit.

### External dependencies
Sometimes it may not be feasible, or even impossible, to store all dependencies
on the blockchain. Examples are p5.js, three.js, python or Blender to name a
few. There is no clearly defined way to describe external dependencies in such a
way that on-chain NFTs can be reproduced by third parties.

## Metadata
The Elpis Metadata Standard builds on the existing
[CIP-0025](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0025)
standard and is divided into three separate entities.

### 1. Scene
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
          "main": <string>,
          "arguments": <object>
        }
      }
    },
    "version": <version_id>
  }
}
```

Additional properties:
- **`renderer`**: an object with two properties
  - **`main`**: the fingerprint of the token containing the actual renderer
  - **`arguments`**: an object with arbitrary key/value pairs used as arguments
    for the invocation of the renderer

### 2. Renderer
The renderer token is stored in the **`files`** property as an embedded
base64-encoded string. It can either be a self-contained program or a program
with external dependencies.

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

        "dependencies": <array | null>
      }
    },
    "version": <version_id>
  }
}
```

Additional properties:
- **`dependencies`**: an optional array of strings with the fingerprints (e.g.
  `asset17pwjm6702q8kstxddv6may3ltlv6gactp666d4`) or names (e.g. `p5.js@1.5.0`)
  of the dependencies.

### 3. Dependency (optional)
A dependency token can be either 

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
        }]
      }
    },
    "version": <version_id>
  }
}
```

## License
Elpis Metadata Standard (v1.0) is licensed under
[CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/legalcode).

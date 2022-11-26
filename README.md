# Elpis Metadata Standard (v1.0)
A metadata standard for storing distributed on-chain NFTs on Cardano.

## Introduction
This standard is useful for generative on-chain art but it may also apply to
other use cases. It aims to solve the three following problems.

### Storage limit
Cardano is very well suited for on-chain NFTs. Compared to other blockchains,
Cardano has the lowest L1 storage cost per kB, but the maximum transaction size
of 16 kB is more limited in comparison to other chains.

### Inefficient use of storage
Some existing on-chain projects on Cardano make inefficient use of block space
by repeatedly storing the same monolithic blob accompanied by a few unique
parameters. This results in thousands of copies of the same code, often close to
or at the full available 16 kB.

### External dependencies
Sometimes it may not be feasible or even impossible to store all dependencies
on-chain. Examples are p5.js, three.js, python or Blender to name a few. There
is no clearly defined way to describe external dependencies in such a way that
on-chain NFTs can be reproduced.

## Definitions

### Scene (Subject/Object/Sketch/Token/Asset...?)


### Renderer
The renderer is an NFT carrying either a self-contained program or one with
dependencies. In the case of the latter, all dependencies are defined in the
metadata of the renderer. The policy 

### Dependency

## Metadata
The Elpis Metadata Standard builds on the existing
[CIP-0025](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0025)
standard and is divided into three separate parts.

### 1. Scene
The **scene** contains all the information to render the NFT.

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
- The **`renderer`** is an object with two properties:
  - **`main`**: the fingerprint of the token containing the renderer
  - **`arguments`**: an object with arbitrary key/value pairs used as arguments
    for the invocation of the renderer

### 2. Renderer
The renderer can either be a self-contained program, stored in the **`files`**
property as a base64 encoded string, or a program with external dependencies.

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

        "dependencies": [
          <string>
        ]
      }
    },
    "version": <version_id>
  }
}
```

Any dependencies should be defined in the **`dependencies`** property, which is
an array of strings.

### 3. Dependency (optional)
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

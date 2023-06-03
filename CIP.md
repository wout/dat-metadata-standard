---
CIP: 98
Title: Metadata Standard for Distributed Artefact Tokens
Category: Metadata
Status: Proposed
Authors:
    - Wout Fierens <hi@wout.codes>
Discussions:
    - https://github.com/cardano-foundation/cips/pulls/?
Created: 2023-06-02
License: CC-BY-4.0
---

## Abstract

This standard is intended for on-chain generative tokens but may also be suitable for other use cases. It describes a method to store code on the blockchain in a distributed and space-efficient way using Cardano's Native Tokens.

Generative tokens created following this standard are called **Distributed Artefact Tokens**, or **DAT**s. They are not necessarily a replacement for NFTs but rather a separate class of tokens in their own right. They can be fungible, semi-fungible or non-fungible. 

DATs introduce a way to instruct viewers to query information from the blockchain. Queries may include information current state of the blockchain, details from the token's mint transaction or details from previously minted tokens. The queried data can be used to create dynamic tokens, evolving tokens, interlinked token collections, etc.

## Motivation: why is this CIP necessary?

The motive for creating this standard is best described through the three following problems.

### Problem 1: Storage limit

Cardano is very well suited for on-chain generative tokens. Compared to other blockchains, it has the lowest L1 storage cost per kB, but the maximum transaction size of 16 kB is more limited compared to other chains.

How can we create generative tokens with larger on-chain codebases without hurting the blockchain?

### Problem 2: Inefficient use of block space

Some existing on-chain projects on Cardano make inefficient use of block space by repeatedly storing the same monolithic blob, accompanied by a few unique parameters. The result is thousands of copies of the same code, close to or at the total capacity of the 16 kB transaction limit.

Without imposing more restrictions on creators, how do we drastically reduce the on-chain storage footprint of generative token collections?

### Problem 3: Dependencies and archival qualities

Storing all dependencies for a generative token on the blockchain is not always convenient, sensible or even viable. Examples are p5.js, three.js, Processing or Blender, to name a few. There is no clearly defined way to describe external dependencies so that digital artefacts can be rendered or reproduced by third parties now, or in the future.

What's the best way to describe external dependencies and maximise archival qualities of web dependencies?

## Specification

The DAT Metadata Standard builds on the existing [CIP-0025](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0025) standard and is divided into three separate entities.

### **1**. Scene

The *scene* token is the part the end user will receive in their wallet. It contains all the information to render the DAT.

#### **1.a.** Metadata

The scene token requires a `renderer` property to be present, referencing the token containing the code of the renderer.

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
          "license": <string>,
          <other_properties>
        }],
        
        "renderer": {
          "main": <asset_name>,
          "arguments": <array>
        },

        "properties": {
          <properties>
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

**Note**: If a DAT uses data from the blockchain to drive its generative algorithm, it's impossible to create a representational thumbnail image before the mint. So the preview image for wallets and marketplaces should be treated more as an album cover than a visual representation of the actual artefact.

#### **1.b.** Argument directives

Several directives for dynamic arguments can be passed to the renderer. Before rendering a DAT, viewers must resolve any directives by querying the required data form the blockchain and pass the actual values to the renderer.

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

Passing argument directives to the renderer is just like static arguments. For example:

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

**Important**: Referencing another token's arguments does not work recursively.

**Note**: The list of directives mentioned above is not exhaustive, nor final. Proposals for new directives will be considered and added over time.

### **2**. Renderer

The *renderer* token is always part of the same `policy_id`. It can either be a self-contained program or one with dependencies. Within the same policy, multiple *renderer* tokens can exist, but *scene* tokens can only reference one at a time.

#### **2.a.** Metadata

The renderer's code is stored in the `files` property as-is or as a base64-encoded string. The `name` property of the file should match the `asset_name` with the appropriate file extenstion, so viewers can filter out the renderer-related files.

Instructions and/or requirements to reproduce the token can be stored in a file named `instructions`. For browser-based artworks, this should include the latest browser version(s) in which the token works. For projects executed locally, it should be a dependency file for a package manager (see attachment `examples/renderer/instructions.md`).

```
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
        }]
      }
    }
  }
}
```

Properties for the *renderer* token:
- **`outputType`** (_required_): the mime type of the renderer's output (it's up to the viewer to define the supported formats)
- **`dependencies`** (_optional_): an array of objects with dependency definitions

While not mandatory, it's advisable to add a **`license`** property to every file in the `files` section. More info on licenses below in section 2.e.

**Note**: The renderer token should be burned after minting to free up the UTxO.

#### **2.b.** Policy dependencies

These are project-specific on-chain dependencies managed by the creator. They must be minted within the same `policy_id`.

```
{
  "type": "onchain",
  "asset_name": <asset_name>
}
```

#### **2.c.** Viewer dependencies:

These are on-chain dependencies managed by the viewer and made available to the *renderer* on execution. They can be referenced by using the dependency's asset `fingerprint`:

```
{
  "type": "internal",
  "fingerprint": <asset_fingerprint>
}
```

Or the asset's `policy_id` and `asset_name`:

```
{
  "type": "internal",
  "policy_id": <policy_id>,
  "asset_name": <asset_name>
}
```

The former being more concise and the latter more readable.

#### **2.d.** External dependencies:

These are off-chain dependencies managed by the viewer and made available to the *renderer* on execution.

```
{
  "type": "external",
  "name": <library_name>,
  "version": <version_number>
}
```

For examples, see attachment `examples/renderer/dependencies.md`.

#### **2.e.** License types

For creators, it is recommended to choose a license that aligns with their values and with the purpose of the digital artefact. Any license can be used, but popular licenses are:

- [NFT License 2.0](https://www.nftlicense.org/)
- [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
- [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- [AGPL 3.0](https://www.gnu.org/licenses/agpl-3.0.en.html)

Using [no license](https://choosealicense.com/no-permission/) is also an option to explicitly indicate that no one other than the creator may use the software.

### **3**. Dependency

There are two diff of *dependency* tokens. The first is the 

 are part of the same `policy_id`. Its code is stored in the **`files`** property as-is or as a base64-encoded string. The `name` property of the file should match the `asset_name`. Similar to the *renderer*, every file can have an individual `license` property.

Dependencies can consist of multiple parts if they don't fit into one 16kB transaction. The dependency referenced from the renderer serves as an entrypoint referencing the additional parts, and should contains the first part as well. Viewers can decide how many parts to allow or support.

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

While not mandatory, it's advisable to add a **`license`** property to each file in the `files` section. More info on licenses in section 2.e.

**Note**: Dependency tokens should be burned after minting to free up the UTxOs.

## Rationale: how does this CIP achieve its goals?

- best described by example
- problem 1
  - separation renderer/dependencies
  - chunked dependencies
- problem 2
  - non-DAT example 1:
    - actual size: 17190 x 5.34kB = 91792kB (~ 90MB)
    - potential size: 1 x 4.91kb + 17190 x 0.53kB = 9116kB (~ 8.9MB)
    - potential savings: ~90.1%
  - non-DAT example 2:
    - actual size: 1744 x 13.9kB = 24241kb (~ 24MB)
    - potential size: 1 x 13.4kB + 1744 x 1.46kB = 2556kB (~ 2.5MB)
    - potential savings: ~89.6%
  - DAT example (TBDs of The Perfect NFT):
    - non-DAT size: 3409 x 25.24kB = 86043kB (~ 84.03MB)
    - actual size: 1 x 10.96kB + 1 x 12.86kB + 3409 x 1.42kB = 4901kB (~ 4.79MB)
    - actual savings: ~94.3%
- problem 3
  

<!-- The rationale fleshes out the specification by describing what motivated the design and what led to particular design decisions. It should describe alternate designs considered and related work. The rationale should provide evidence of consensus within the community and discuss significant objections or concerns raised during the discussion.

It must also explain how the proposal affects the backward compatibility of existing solutions when applicable. If the proposal responds to a CPS, the 'Rationale' section should explain how it addresses the CPS, and answer any questions that the CPS poses for potential solutions.
-->

## Path to Active

### Acceptance Criteria
<!-- Describes what are the acceptance criteria whereby a proposal becomes 'Active' -->

### Implementation Plan

- one implementation alreaday live (venster)
- desktop viewer available later 2023 (open-source) 
- working with existing viewers to implement the DMS
- provide open source tooling 

## Copyright

[CC-BY-4.0]: https://creativecommons.org/licenses/by/4.0/legalcode

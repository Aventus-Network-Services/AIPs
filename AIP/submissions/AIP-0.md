---
aip: 0
title: AIP Purpose and Guidelines
status: Active
type: Informational
author: Emmanuel Ngubo (@dr-ngubo)
created: 2022-04-05
---

## What is an AIP?

Aventus Improvement Proposals (AIP) describes standards for the Aventus Ecosystem, including core protocol specifications and processes. These AIPs are technical communication processed off-chain and recognised formally. This is not a substitute for the governance process run here but it is a collection of sound and forward-thinking solutions and functionalities for the network. However, they do not represent a commitment of any form towards existing projects.

## AIP Rationale

The intention here is for AIPs to be the primary mechanism for proposing solutions and new functionality by the community for the Aventus Network, and for documenting the design decisions that have gone into Aventus.

For Aventus implementers, AIPs are a convenient way to track the progress of their implementation.

## AIP Types

There are five types of AIP:

- Core - Improvements to the components of the substrate build of the Aventus Network such as pallets and any other changes that affect the blockchain protocol. This includes changes to the block or transaction validity rules and other network protocol changes.

- Layer 1 - Improvements to the smart contracts on Layer 1.

- Interface - Improvements to the communication mechanism between the Aventus blockchain and other blockchain networks such as Ethereum, Polkadot, etc. This also includes improvements around the client.

- Ecosystem - Improvements to the various tools built on the network such as Explorer, Wallet, Governance, etc.

- Informational - describes an Aventus design issue, or provides general guidelines or information to the Aventus community, but does not propose a new feature.

To understand more about the current state of the Aventus network, check out our [whitepaper](https://github.com/AventusProtocolFoundation/docs/blob/master/resources/Aventus%20Network%20Technical%20Whitepaper.pdf).

An AIP must meet certain minimum criteria. It must be a clear and complete description of the proposed enhancement. The enhancement must represent a net improvement.

## AIP Work Flow

### Shepherding an AIP

Parties involved in the process are you, the *AIP author*, the AIP editors, and the Aventus Core Developers.

Before you begin writing a formal AIP, you should vet your idea. Ask the Aventus community first if an idea is original to avoid wasting time on something that will be rejected based on prior research. It is thus recommended to open a discussion thread on [the Aventus Telegram channel](https://t.me/Aventus_Official) to do this.

### AIP Process

The following is the standardization process for all AIPs in all tracks:

- **Draft** - The idea has been formally accepted on the github repository by merging the draft into the submissions folder.

- **Community review** - The submitted proposal will be announced on various channels for 2 weeks OR The submitted proposal will be voted on by AVT holders. The result of the vote will either be accept or reject.

- **Proposed** - This has been tested in a sandbox environment and is believed to be working. Also, an approved plan to implement this on the mainnet has been constructed and accepted. Further changes are unlikely at this stage.

- **Active** - The proposal is deemed to be working and has passed all checks and is now deployed on the Aventus mainnet.

- **On Hold** - Work on this has been halted either due to the author/submitted putting the proposal on hold or a decision was made to halt it.

- **Obsolete** - A more recent proposal supersedes it in capabilities and value.

- **Rejected** - Concerns have been raised around the proposal.

## AIP Formats and Templates

AIPs should be written in [markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) format. There is a [template](../template.md) to follow.

## AIP Header Preamble

Each AIP must begin with an [RFC 822](https://www.ietf.org/rfc/rfc822.txt) style header preamble, preceded and followed by three hyphens (`---`). This header is also termed ["front matter" by Jekyll](https://jekyllrb.com/docs/front-matter/). The headers must appear in the following order.

`AIP`: *AIP number* (this is determined by the AIP editor)

`title`: *The AIP title is a few words, not a complete sentence*

`description`: *Description is one full (short) sentence*

`author`: *The list of the author's or authors' name(s) and/or username(s), or name(s) and email(s). Details are below.*

`status`: *Draft, Community Review, Proposed, Active, On Hold, Obsolete, Rejected*

`type`: *One of `Core`, `Layer1`, or `Ecosystem`, `Interface`, or `Informational`*

`created`: *Date the AIP was created on*

`requires`: *AIP number(s)* (Optional field)

Headers that permit lists must separate elements with commas.

Headers requiring dates will always do so in the format of ISO 8601 (yyyy-mm-dd).

#### `author` header

The `author` header lists the names, email addresses or usernames of the authors/owners of the AIP. Those who prefer anonymity may use a username only, or a first name and a username. The format of the `author` header value must be:

> Random J. User &lt;address@dom.ain&gt;

or

> Random J. User (@username)

if the email address or GitHub username is included, and

> Random J. User

if the email address is not given.

It is not possible to use both an email and a GitHub username at the same time. If important to include both, one could include their name twice, once with the GitHub username, and once with the email.

At least one author must use a GitHub username, in order to get notified on change requests and have the capability to approve or reject them.

#### `type` header

The `type` header specifies the type of AIP: Core, Layer1, Interface, Ecosystem, or Informational.

#### `created` header

The `created` header records the date that the AIP was assigned a number. Both headers should be in yyyy-mm-dd format, e.g. 2001-08-14.

#### `requires` header

AIPs may have a `requires` header, indicating the AIP numbers that this AIP depends on.

## Linking to External Resources

Links to external resources **SHOULD NOT** be included. External resources may disappear, move, or change unexpectedly.

## Linking to other AIPs

References to other AIPs should follow the format `AIP-N` where `N` is the AIP number you are referring to.  Each AIP that is referenced in an AIP **MUST** be accompanied by a relative markdown link the first time it is referenced, and **MAY** be accompanied by a link on subsequent references.  The link **MUST** always be done via relative paths so that the links work in this GitHub repository, forks of this repository, the main AIPs site, mirrors of the main AIP site, etc.  For example, you would link to this AIP with `[AIP-1](./AIP-1.md)`.

## Auxiliary Files

Images, diagrams and auxiliary files should be included in a subdirectory of the `assets` folder for that AIP as follows: `assets/AIP-N` (where **N** is to be replaced with the AIP number). When linking to an image in the AIP, use relative links such as `../assets/AIP-1/image.png`.

## Transferring AIP Ownership

It occasionally becomes necessary to transfer ownership of AIPs to a new *champion*. In general, we'd like to retain the original author as a co-author of the transferred AIP, but that's really up to the original author. A good reason to transfer ownership is because the original author no longer has the time or interest in updating it or following through with the AIP process, or has fallen off the face of the 'net (i.e. is unreachable or isn't responding to email). A bad reason to transfer ownership is because you don't agree with the direction of the AIP. We try to build consensus around an AIP, but if that's not possible, you can always submit a competing AIP.

If you are interested in assuming ownership of an AIP, send a message asking to take over, addressed to both the original author and the AIP editor. If the original author doesn't respond to the email in a timely manner, the AIP editor will make a unilateral decision (it's not like such decisions can't be reversed :).

## AIP Editor Responsibilities

For each new AIP that comes in, an editor does the following:

- Read the AIP to check if it is ready: sound and complete. The ideas must make technical sense, even if they don't seem likely to get to final status.
- The title should accurately describe the content.
- Check the AIP for language (spelling, grammar, sentence structure, etc.), markup (GitHub flavored Markdown), code style

If the AIP isn't ready, the editor will send it back to the author for revision, with specific instructions.

Once the AIP is ready for the repository, the AIP editor will:

- Assign an AIP number (generally the PR number, but the decision is with the editors)
- Merge the corresponding [pull request](https://github.com/Aventus/AIPs/pulls)
- Send a message back to the AIP author with the next step.

Many AIPs are written and maintained by developers with write access to the Aventus codebase. The AIP editors monitor AIP changes, and correct any structure, grammar, spelling, or markup mistakes we see.

## Style Guide

### AIP numbers

When referring to an AIP by number, it should be written in the hyphenated form `AIP-X` where `X` is the AIP's assigned number.

### RFC 2119

AIPs are encouraged to follow [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt) for terminology and to insert the following at the beginning of the Specification section:

> The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

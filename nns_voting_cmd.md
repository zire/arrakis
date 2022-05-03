# Vote on NNS Proposals with Command-Line Tool

## Table of Contents

- [Background](#background)
- [How](#how)
- [Next](#next)
- [References](#references)

## Background

This guide explains how to cast votes for your staked neurons on Network Nervous System ("NNS"), the DAO that governs the operation and administration of the Internet Computer ("IC") network completely through command-line tools of *quill* and *dfx*, without ever going through a web interface.

As a [high-frequence user of NNS](https://twitter.com/herbertyang/status/1512623623672328194), I've run into challenges in the current NNS front-end app [https://nns.ic0.app/](https://nns.ic0.app/). It logs me out every 10 minutes. This is a sound security measure, but it makes my workflow very slow and inefficient. The recent flurry of spam proposals is not helping either. DFINITY's engineering team recently launched v2 for the official NNS front-end app and refactored the previous *Flutter* framework with *Svelte*, which gives the site a much needed boost in loading speed. However, the "Accounts", "Neurons" and "Canisters" tabs are still in Flutter so the full power of Svelte is yet to manifest. 

So I need a method to review pending NNS proposals and cast votes on them fast and without interruption. The entire set of interface APIs for NNS governance is already categorically defined in [governance.did](https://raw.githubusercontent.com/dfinity/ic/master/rs/nns/governance/canister/governance.did). Technically, somebody could develop another NNS front-end app completely based off this one single file (which would be a worthy hackathon bounty). 

Two command-line tools are needed. [*quill*](https://github.com/dfinity/quill) is a minimalistic tool created by DFINITY to manage neurons through air-gaged cold wallets. [*dfx*](https://github.com/dfinity/sdk) is the official SDK developed by DFINITY for building apps on the Internet Computer. Theoretically, the entire workflow can be done in *dfx* and probably should be. For now I'll still go with the hybrid approach as *quill* makes a few steps slightly easier.

I was very buoyed by [Mix Labs](https://twitter.com/MixLabs_) developer [Liquan](https://twitter.com/liquan_eth)'s [earlier experiment on this topic in July 2021](https://forum.dfinity.org/t/how-to-use-dfx-to-interact-with-nns-canisters-instead-of-nns-app/6013) and drew many inspirations from his scripts. My DFINITY colleagues [Paul](https://twitter.com/paulliuicp) provided constant guidance along the way and [David](https://twitter.com/daviddalbusco) provided the last piece to complete the puzzle.


## How

## Next

## References

- [How to use dfx to interact with NNS canisters instead of nns app](https://github.com/flyq/blogs/blob/master/Dfinity/How%20to%20use%20dfx%20to%20interact%20with%20NNS%20canisters%20instead%20of%20nns%20app_en.md#how-to-use-dfx-to-interact-with-nns-canisters-instead-of-nns-app) by [Liquan](https://twitter.com/liquan_eth) of [Mix Labs](https://twitter.com/MixLabs_)
- [quill, minimalistic ledger and governance toolkit for cold wallets](https://github.com/dfinity/quill#:~:text=quill%20--pem-file%20%3Cpath%3E%20neuron-stake%20--amount%202.5%20--name%201,online%20machine%20using%20the%20send%20command%20from%20above), by [DFINITY](https://twitter.com/dfinity)
- [ICP staking with seed phrase and air-gapped computer](https://wiki.internetcomputer.org/wiki/ICP_staking_with_seed_phrase_and_air-gapped_computer) by [DFINITY](https://twitter.com/dfinity)
- [Introducing Quill, a Ledger and Governance Toolkit for the Internet Computer](https://medium.com/dfinity/introducing-quill-a-ledger-and-governance-toolkit-for-the-internet-computer-1df086ce5642) by [DFINITY](https://twitter.com/dfinity)
- [NNS API](https://raw.githubusercontent.com/dfinity/ic/master/rs/nns/governance/canister/governance.did) by [DFINITY](https://twitter.com/dfinity)
- [NNS API Governance Parameters](https://github.com/dfinity/ic/blob/master/rs/nns/governance/proto/ic_nns_governance/pb/v1/governance.proto) by [DFINITY](https://twitter.com/dfinity)

Herbert Yang, May 2022
# Vote on NNS Proposals with Command-Line Tool

## Table of Contents

- [Background](#background)
- [Why](#why)
- [Disclaimer](#disclaimer)
- [How](#how)
- [Next](#next)
- [References](#references)

## Background

This guide explains how to cast votes for your staked neurons on Network Nervous System ("NNS"), the DAO that governs the operation and administration of the Internet Computer ("IC") network **completely through command-line tools of *quill* and *dfx*, without ever going through a web interface**.

As a [high-frequence user of NNS](https://twitter.com/herbertyang/status/1512623623672328194), I've run into challenges in the current NNS front-end app [https://nns.ic0.app/](https://nns.ic0.app/). It logs me out every 10 minutes. This is a sound security measure, but it makes my workflow very slow and inefficient. The recent flurry of spam proposals is not helping either. DFINITY's engineering team recently launched v2 for the official NNS front-end app and refactored the previous *Flutter* framework with *Svelte*, which gives the site a much needed boost in loading speed. However, the "Accounts", "Neurons" and "Canisters" tabs are still in Flutter so the full power of Svelte is yet to manifest. 

So I need a method to review pending NNS proposals and cast votes on them fast and without interruption. The entire set of interface APIs for NNS governance is already categorically defined in [governance.did](https://raw.githubusercontent.com/dfinity/ic/master/rs/nns/governance/canister/governance.did). Technically, somebody could develop another NNS front-end app completely based off this one single file (which would be a worthy hackathon bounty). 

Two command-line tools are needed. [*quill*](https://github.com/dfinity/quill) is a minimalistic tool created by DFINITY to manage neurons through air-gagged cold wallets. [*dfx*](https://github.com/dfinity/sdk) is the official SDK developed by DFINITY for building apps on the Internet Computer. Theoretically, the entire workflow can be done in *dfx* and probably should be. For now I'll still go with the hybrid approach as *quill* makes a few steps slightly easier.

I was very buoyed by [Mix Labs](https://twitter.com/MixLabs_) developer [Liquan](https://twitter.com/liquan_eth)'s [earlier experiment on this topic in July 2021](https://forum.dfinity.org/t/how-to-use-dfx-to-interact-with-nns-canisters-instead-of-nns-app/6013) and drew many inspirations from his scripts. My DFINITY colleagues [Paul](https://twitter.com/paulliuicp) provided constant guidance along the way and [David](https://twitter.com/daviddalbusco) provided the last piece to complete the puzzle.

## Why

For IC developers, this how-to guide could serve as a baseline reference that might spark a few ideas on how you can build your very own NNS front-end. DFINITY strives to develop the best infrastructure possible for IC developers to refactor the entire Internet. It would very much prefer to leave the application-level development to IC developers. There is no sacred cow. If your needs are not met or you are not satisfied with the current stack, build one yourself. That's the spirit of the Internet Computer. 

For IC holders that are not developers, this article could serve as another validation point that speaks to the continuous decentralization happening on the Internet Computer. DFINITY's R&D team has iterated IC 4 times in the last 5 years before Genesis launch. Among the many paradigm-shifting innovations that will re-shape the Internet as we know it, one of them is this [350-line file](https://raw.githubusercontent.com/dfinity/ic/master/rs/nns/governance/canister/governance.did) that defines the entire working logic of NNS, the most advanced Decentralized Autonomous Organization ("DAO") in crypto. It's there for anyone curious enough to tinker with and build applications around.

## Disclaimer

>YOU EXPRESSLY ACKNOWLEDGE AND AGREE THAT USE OF THIS METHOD IS AT YOUR SOLE RISK. THE AUTHOR OF THIS ARTICLE SHALL NOT BE LIABLE FOR DAMAGES OF ANY TYPE, WHETHER DIRECT OR INDIRECT.

If you're not comfortable with command-line tools, this guide is not for you. Actions executed by *quill* and *dfx* could trigger irreversible events for your staked neurons on NNS and impact your staking rewards. Please conduct your own research and experiment before trying to follow this guide.

This how-to guide describes my personal workflow that suits my unique needs on NNS governance. Most ICP holders just follow named neurons and rarely need to worry about this level of technical details. This shall not be considered an "official" guide by DFINITY that encourages IC developers to follow its footsteps. It's more for my own personal reference rather than promoting a new way of doing things in the IC community at large.

## How

At a high level, this workflow involves 3 steps. First, use *quill* to create a new neuron for staking on the IC ledger. Second, use *dfx* to review pending NNS proposals and cast votes. Third, use *dfx* to assign following topics and followees.

I'm on *macOS Monterey 12.3.1*, *MacBook Pro*, and *dfx 0.9.2*. 

Let's denote [nns.ic0.app](https://nns.ic0.app) as *Official NNS Front-End App*. 

### Step 1 - install quill and dfx

Download the binary file [quill-macos-x86_64](https://github.com/dfinity/quill/releases/download/test-release/quill-macos-x86_64) for [quill](https://github.com/dfinity/quill).

Move the downloaded file to a folder you prefer, say `/Applications` and rename the file to be `quill` for ease of use.

Give this *quill* executable file proper permission to enable write access for its owner. 

```
$ chmod 755 quill
```

*quill* is ready for use now. When in folder `/Applications`, launch the app

```
$ ./quill --help
```
and here's the main menu

```
quill 0.2.15

Ledger & Governance ToolKit for cold wallets

USAGE:
    quill [OPTIONS] <SUBCOMMAND>

OPTIONS:
    -h, --help                         Print help information
        --hsm
        --hsm-id <HSM_ID>
        --hsm-libpath <HSM_LIBPATH>
        --hsm-slot <HSM_SLOT>
        --pem-file <PEM_FILE>          Path to your PEM file (use "-" for STDIN)
        --qr                           Output the result(s) as UTF-8 QR codes
        --seed-file <SEED_FILE>        Path to your seed file (use "-" for STDIN)
    -V, --version                      Print version information

SUBCOMMANDS:
    account-balance         Queries a ledger account balance
    claim-neurons           Claim seed neurons from the Genesis Token Canister
    generate                Generate a mnemonic seed phrase and generate or recover PEM
    get-neuron-info
    get-proposal-info
    help                    Print this message or the help of the given subcommand(s)
    list-neurons            Signs the query for all neurons belonging to the signing principal
    list-proposals
    neuron-manage           Signs a neuron configuration change
    neuron-stake            Signs topping up of a neuron (new or existing)
    public-ids              Prints the principal id and the account id
    qr-code                 Print QR code for data e.g. principal id
    scanner-qr-code         Print QR Scanner dapp QR code: scan to start dapp to submit QR
                            results
    send                    Sends a signed message or a set of messages
    transfer                Signs an ICP transfer transaction
    update-node-provider    Update node provider details

```

Install *dfx* from [smartcontracts.org](https://smartcontracts.org/docs/developers-guide/install-upgrade-remove.html), which is owned by DFINITY. 

```
$ sh -ci "$(curl -fsSL https://smartcontracts.org/install.sh)"
```

Check if the dfx package is properly installed

```
$ dfx help
```

and here's the main menu

```
dfx 0.9.2
The DFINITY Executor

USAGE:
    dfx [OPTIONS] <SUBCOMMAND>

OPTIONS:
    -h, --help                   Print help information
        --identity <IDENTITY>
        --log <LOGMODE>          [default: stderr] [possible values: stderr, tee, file]
        --logfile <LOGFILE>
    -q, --quiet
    -v, --verbose
    -V, --version                Print version information

SUBCOMMANDS:
    bootstrap    Starts the bootstrap server
    build        Builds all or specific canisters from the code in your project. By default, all
                 canisters are built
    cache        Manages the dfx version cache
    canister     Manages canisters deployed on a network replica
    config       Configures project options for your currently-selected project
    deploy       Deploys all or a specific canister from the code in your project. By default,
                 all canisters are deployed
    generate     Generate type declarations for canisters from the code in your project
    help         Print this message or the help of the given subcommand(s)
    identity     Manages identities used to communicate with the Internet Computer network.
                 Setting an identity enables you to test user-based access controls
    ledger       Ledger commands
    new          Creates a new project
    ping         Pings an Internet Computer network and returns its status
    remote       Commands used to work with remote canisters
    replica      Starts a local Internet Computer replica
    start        Starts the local replica and a web server for the current project
    stop         Stops the local network replica
    toolchain    Manage the dfx toolchains
    upgrade      Upgrade DFX
    wallet       Helper commands to manage the user's cycles wallet
```

### Step 2 - create a new neuron for staking and voting

There are two ways a staking neuron can be created, either on *Official NNS Front-End App* that's based on a web interface or through *dfx* the command-line SDK tool. My ICPs are staked in a few neurons that were created on *Official NNS Front-End App*, which is authenticated by an *Internet Identity* account that I created on [https://identity.ic0.app](https://identity.ic0.app). 

Here's the catch - *quill* was created to handle offline, air-gagged cold wallets to interact with the IC ledger. It's designed so that all its commands will be output into a QR code that can be scanned by IC ledger. It locates the right neuron by looking up the *PEM* file in the local machine that contains the **private key** for the related *identity* (this *identity* is NOT to be confused with *Internet Identity*) created by *dfx*. Accounts created on web-based *Internet Identity* has no *PEM* file (private key shall exist only on local machine, never on the web). Therefore, *quill* CANNOT handle neurons related to accounts created on *Internet Identity*, which are the neurons you will see from *Official NNS Front-End App*. 

That's a bummer! How do I ever direct the voting of my existing neurons on command-line then? 

One solution is to create a new neuron from *quill* based on *identity* created by *dfx* on the local machine; do all the NNS governance on this neuron; and set all my existing neurons (created on the web-interfaced *Official NNS Front-End App*) to follow this neuron. Let's try this.

You can create multiple *identity* with *dfx* on your local machine. I have an *identity* that handles all my canisters and a separate *identity* to handle NNS governance. 

List all the current identities

```
$ dfx identity list
```

Set the one that shall be used for NNS governance, say `id-nns`

```
$ dfx identity use id-nns
```

Verify that `id-nns` is the current identity assumed by *dfx*

```
$ dfx identity whoami
```

The *PEM* file is stored here `~/.config/dfx/identity/id-nns/identity.pem`

Grab the public IDs (both Account ID and Principal ID) for this identity (this cannot be obtained from the web interface *Official NNS Front-End App*) with subcommand `public-ids`.

```
$ ./quill --pem-file ~/.config/dfx/identity/id-nns/identity.pem public-ids
```

*quill* shall display this for *identity* `id-nns`.

```
Principal id: ytg23-rrskd-bnz5m-66dk2-rqt6w-ilvbq-56aha-ipaue-22e2c-uvjma-vae
Account id: 99f6ab276a23f3308641a06c9b24f3020849ee23774c6196c49e6bebf39e1734
```

Check out the ICP balance for this account with subcommand `account-balance`

```
$ ./quill account-balance 99f6ab276a23f3308641a06c9b24f3020849ee23774c6196c49e6bebf39e1734
```
and it returns

```
Sending message with

  Call type:   query
  Sender:      2vxsx-fae
  Canister id: ryjl3-tyaaa-aaaaa-aaaba-cai
  Method name: account_balance_dfx
  Arguments:   (
  record {
    account = "99f6ab276a23f3308641a06c9b24f3020849ee23774c6196c49e6bebf39e1734";
  },
)
Response: (record { e8s = 990_000 : nat64 })
```
1 ICP equals `100 million` *e8s*. So `990,000` *e8s* would be `0.0099` ICP.

To create a neuron that is eligible for voting on NNS, the neuron needs to have at least `1` ICP with staking period of `6` months. 

So we will need to first top up the ICP balance of this account `1734`. This is a one-time action. I just did that with one of my [Plug](https://twitter.com/plug_wallet) wallets and added `1` ICP into account `1734`. It could have been done in *dfx* too, for command-line maxis. 

Commands in *quill* cannot be executed without running `quill send` first. We will need to pipe the result of `quill neuron-stake` into a JSON message that will be read by `quill send` and sent over to the IC ledger. Note that, this workflow will break the air-gag and the cold wallet will no longer stay cold. I can jump through a few additional hoops to maintain the air-gag but that's not the concern for this guide.

Let's release the kraken with subcommand `neuron-stake`, followed by `send --yes -`.

```
$ ./quill --pem-file ~/.config/dfx/identity/id-nns/identity.pem neuron-stake --amount 1 --name "neuron01" | ./quill send --yes -
```
and it returns

```
- The request is being processed...
- The request is being processed...
- (
    - record {
        - result = opt variant {
            - NeuronId = record { id = 5_006_161_079_443_216_280 : nat64 }
        - };
    - },
```

A new neuron with ID `5_006_161_079_443_216_280` or [5006161079443216280](https://dashboard.internetcomputer.org/neuron/5006161079443216280) was just created. Take down this neuron ID. Don't lose it.

Take a closer look at this neuron with `get-neuron-info` subcommand

```
$ ./quill get-neuron-info 5006161079443216280
```

and it returns

```
Sending message with

  Call type:   query
  Sender:      2vxsx-fae
  Canister id: rrkah-fqaaa-aaaaa-aaaaq-cai
  Method name: get_neuron_info
  Arguments:   (5_006_161_079_443_216_280 : nat64)
Response: (
  variant {
    Ok = record {
      dissolve_delay_seconds = 252_460_800 : nat64;
      recent_ballots = vec {
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_849 : nat64 };
        };
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_846 : nat64 };
        };
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_843 : nat64 };
        };
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_839 : nat64 };
        };
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_832 : nat64 };
        };
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_831 : nat64 };
        };
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_821 : nat64 };
        };
        record {
          vote = 2 : int32;
          proposal_id = opt record { id = 57_818 : nat64 };
        };
        record {
          vote = 2 : int32;
          proposal_id = opt record { id = 57_819 : nat64 };
        };
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_820 : nat64 };
        };
      };
      created_timestamp_seconds = 1_651_465_517 : nat64;
      state = 1 : int32;
      stake_e8s = 100_000_000 : nat64;
      retrieved_at_timestamp_seconds = 1_651_548_764 : nat64;
      voting_power = 200_032_644 : nat64;
      age_seconds = 82_414 : nat64;
    }
  },
)
```

From `stake_e8s = 100_000_000 : nat64;`, it can be verified that this neuron has been staked with exactly `1` ICP as intended.

Now let's set the dissolve delay so that this neuron can be eligible for voting with `neuron-manage` subcommand.

```
$ ./quill --pem-file ~/.config/dfx/identity/id-nns/identity.pem neuron-manage -a $((366*8*24*3600)) 5_006_161_079_443_216_280 | ./quill send --yes -
```

Staking on IC is capped at 8 years. Dissolve Delay is expressed in seconds in *quill*, so a Shell command `$((366*8*24*3600))` is used to hit that 8 year cap. If `365` is used in the formula, the result will be smaller than the maximum cap. Either way, it doesn't matter as long as it's more than 6 months.

From the same output message from `get-neuron-info`, it can be verified that the dissolve delay has been set correctly at `252,460,800` seconds, aka, 8 years.

```
dissolve_delay_seconds = 252_460_800 : nat64;
```

Note that this neuron can only vote on NNS proposals submitted AFTER its creation, not before.

### Step 3 - review NNS proposals and cast votes

Running *dfx* requires several files such as `dfx.json` and `canister_ids.json` for a minimum deployment folder. [Paul](https://github.com/ninegua/) created a simple package. Download it from [here](https://gist.github.com/ninegua/6fa1863837556766e0bcfd8b3aeb4c30). 

Unzip the file, move it into an empty folder (say, `nns-make`), move the folder to `/Applications` folder. Enter into the folder and run `make` in it.

```
cd nns-make
make
```
A few files will be created and downloaded. The folder `nns-make` now looks like

```
.dfx
Makefile
canister_ids.json
dfx.json
governance.did
```

Test run to see if you can successfully query IC ledger

```
$ dfx canister --network=ic call nns list_known_neurons
```

It shall return

```
(
  record {
    known_neurons = vec {
      record {
        id = opt record { id = 4_966_884_161_088_437_903 : nat64 };
        known_neuron_data = opt record {
          name = "ICP Maximalist Network";
          description = opt "The ICPMN neuron will be representative of the end users, investors, developers, project leaders, Dfinity and ICA members, and other IC ecosystem contributors who participate in our community. A primary objective will be to ensure that our neuron can be trusted to always vote on all proposals. The neuron will be configured to follow our elected voting members on all Governance proposals and to follow Dfinity Foundation (DF) and/or Internet Computer Association (ICA) on non-Governance proposals.  \n\nOwnership and configuration of this community neuron as well as voting member expectations are described in the policy document published at https://www.ic.community/followee-neuron-for-icp-maximalist-network/ .  \n\nYou can join the active ICP Maximalist Network community on Telegram at https://t.me/icpmaximalistnetwork ";
        };
      };
      record {
        id = opt record { id = 14_231_996_777_861_930_328 : nat64 };
        known_neuron_data = opt record {
          name = "ICDevs.org";
          description = opt "ICDevs.org is a non-profit that seeks to provide the general public with community organization, educational resources, funding, and scientific discovery for the use and development of the Internet Computer and related technologies. We aim to weigh the interests of developers in the Internet Computer ecosystem. Details of how we vote and how you can participate are found at https://icdevs.org/nns.html";
        };
      };
      record {
        id = opt record { id = 5_967_494_994_762_486_275 : nat64 };
        known_neuron_data = opt record {
          name = "cycledao.xyz";
          description = opt "cycle_dao is a group of Internet Computer ecosystem members who deliberate on proposals and vote via a DAO that controls the cycle_dao neuron. We aim to weigh the interests of all parties in the Internet Computer ecosystem and support the future stability and longevity of the Internet Computer. ";
        };
      };
    };
  },
)
```

In this folder, check out the file [governance.did](https://github.com/dfinity/ic/raw/master/rs/nns/governance/canister/governance.did) that defines the entire API interface for NNS. The last section is probably the most important one, especially with `manage_neuron` command.

```
  claim_gtc_neurons : (principal, vec NeuronId) -> (Result);
  claim_or_refresh_neuron_from_account : (ClaimOrRefreshNeuronFromAccount) -> (
      ClaimOrRefreshNeuronFromAccountResponse,
    );
  get_build_metadata : () -> (text) query;
  get_full_neuron : (nat64) -> (Result_2) query;
  get_full_neuron_by_id_or_subaccount : (NeuronIdOrSubaccount) -> (
      Result_2,
    ) query;
  get_monthly_node_provider_rewards : () -> (Result_3);
  get_network_economics_parameters : () -> (NetworkEconomics) query;
  get_neuron_ids : () -> (vec nat64) query;
  get_neuron_info : (nat64) -> (Result_4) query;
  get_neuron_info_by_id_or_subaccount : (NeuronIdOrSubaccount) -> (
      Result_4,
    ) query;
  get_node_provider_by_caller : (null) -> (Result_5) query;
  get_pending_proposals : () -> (vec ProposalInfo) query;
  get_proposal_info : (nat64) -> (opt ProposalInfo) query;
  list_known_neurons : () -> (ListKnownNeuronsResponse) query;
  list_neurons : (ListNeurons) -> (ListNeuronsResponse) query;
  list_node_providers : () -> (ListNodeProvidersResponse) query;
  list_proposals : (ListProposalInfo) -> (ListProposalInfoResponse) query;
  manage_neuron : (ManageNeuron) -> (ManageNeuronResponse);
  transfer_gtc_neuron : (NeuronId, NeuronId) -> (Result);
  update_node_provider : (UpdateNodeProvider) -> (Result);
```

Get a list of pending proposals with `get_pending_proposals`

```
$ dfx canister --network=ic call nns get_pending_proposals
```

Get detailed info of a specific proposal with its Proposal ID via `get_proposal_info`

```
$ dfx canister --network=ic call nns get_proposal_info 57849
```

Get detailed info of a neuron with `get_neuron_info`, which gives the same outcome as you'd expect from using *quill*.

```
$ dfx canister --network=ic call nns get_neuron_info 5006161079443216280
```

Use `dfx identity` to switch to the designated *identity* for NNS voting `id-nns`.

`1` is support; `2` is against; `0` is not voted yet, based on this [reference file on NNS governance API](https://github.com/dfinity/ic/blob/master/rs/nns/governance/proto/ic_nns_governance/pb/v1/governance.proto)

Cast my `against` vote on proposal `57818`, with `manage_neuron`. This command will be used most often going forward.

```
dfx canister --network=ic call nns manage_neuron "(record {id=opt record{id=5_006_161_079_443_216_280:nat64};command=opt variant{RegisterVote=record {vote=2:int32;proposal=opt record{id=57_818:nat64}}}})"
```

Run `get_neuron_info` again to check if the voting has been effective

```
$ dfx canister --network=ic call nns get_neuron_info 5006161079443216280
```

Verify that my neuron's vote has been recorded in the output message

```
(
  variant {
    Ok = record {
      dissolve_delay_seconds = 252_460_800 : nat64;
      recent_ballots = vec {
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_849 : nat64 };
        };
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_846 : nat64 };
        };
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_843 : nat64 };
        };
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_839 : nat64 };
        };
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_832 : nat64 };
        };
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_831 : nat64 };
        };
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_821 : nat64 };
        };
        record {
          vote = 2 : int32;
          proposal_id = opt record { id = 57_818 : nat64 };
        };
        record {
          vote = 2 : int32;
          proposal_id = opt record { id = 57_819 : nat64 };
        };
        record {
          vote = 1 : int32;
          proposal_id = opt record { id = 57_820 : nat64 };
        };
      };
      created_timestamp_seconds = 1_651_465_517 : nat64;
      state = 1 : int32;
      stake_e8s = 100_000_000 : nat64;
      joined_community_fund_timestamp_seconds = null;
      retrieved_at_timestamp_seconds = 1_651_557_629 : nat64;
      known_neuron_data = null;
      voting_power = 200_036_155 : nat64;
      age_seconds = 91_279 : nat64;
    }
  },
)
```

Success!

### Step 4 - set follow topics and followees in NNS

Step 3 can be repeated as frequently as needed, with a different *Proposal ID* each time.

We also need to set up follow topic and followees, so that we can focus on the most consequential proposals rather than routine updates. For now I'm following Neuron 27 (DFINITY) and Neuron 28 (Internet Computer Association) for all exchange related proposals, which is `topic 2` according to the [reference file on NNS governance API](https://github.com/dfinity/ic/blob/master/rs/nns/governance/proto/ic_nns_governance/pb/v1/governance.proto). The NNS way is to pick a topic first, then neuron IDs. 

```
  Unspecified = 0,
  ManageNeuron = 1,
  ExchangeRate = 2,
  NetworkEconomics = 3,
  Governance = 4,
  NodeAdmin = 5,
  ParticipantManagement = 6,
  SubnetManagement = 7,
  NetworkCanisterManagement = 8,
  Kyc = 9,
  NodeProviderRewards = 10,
```

This step can be accomplished in *dfx* or *quill*. *quill* has easier syntax (thought takes a bit of guestwork).

For every topic, if only one neuron will be followed

```
$ ./quill --pem-file ~/.config/dfx/identity/id-nns/identity.pem neuron-manage --follow-topic=2 --follow-neurons=27 5006161079443216280 |./quill send --yes -
```

If multiple neurons will be followed, the flag `--follow-neurons` must be placed in the back AFTER *neuron ID*.

```
$ ./quill --pem-file ~/.config/dfx/identity/id-nns/identity.pem neuron-manage 5006161079443216280 --follow-topic=2 --follow-neurons 27 28 | ./quill send --yes -
```

It returns

```
Sending message with

  Call type:   update
  Sender:      ytg23-rrskd-bnz5m-66dk2-rqt6w-ilvbq-56aha-ipaue-22e2c-uvjma-vae
  Canister id: rrkah-fqaaa-aaaaa-aaaaq-cai
  Method name: manage_neuron
  Arguments:   (
  record {
    id = opt record { id = 5_006_161_079_443_216_280 : nat64 };
    command = opt variant {
      Follow = record {
        topic = 2 : int32;
        followees = vec {
          record { id = 27 : nat64 };
          record { id = 28 : nat64 };
        };
      }
    };
    neuron_id_or_subaccount = null;
  },
)
Request ID: 0x96ac0b3c29855507dee2e615df8eaa6eb0065b64b59bb49d5b60db40b68e4113
The request is being processed...
The request is being processed...
(record { command = opt variant { Follow = record {} } })
```

Verify that this following relationship has been established with `get_full_neuron` in `dfx`. The previous subcommand `get_neuron_info` would not provide this detail.

```
$ dfx canister --network=ic call nns get_full_neuron 5006161079443216280
```

The output message contains this, verifying that the follow action has been effective.

```
      followees = vec {
        record {
          2 : int32;
          record {
            followees = vec {
              record { id = 27 : nat64 };
              record { id = 28 : nat64 };
            };
          };
        };
        record {
          0 : int32;
          record { followees = vec { record { id = 28 : nat64 } } };
        };
      };
```

This completes the entire loop. No more auto logout every ten minutes. No more waiting for webpage loading. It's entirely run on command line. You can control the entire workflow at your own pace with probably more info than you actually need.

## Next

This command-line based workflow can be iterated in a few directions:

1. Replace all *quill* commands with *dfx*, which is more canonical with more robust syntax support.
2. Build a *Svelte* front-end just for NNS voting and community funding, and keep it separate from canister management and ICP transfer. Arguably both canister management and ICP transfer can probably benefit from having their own independent apps, and be developed by the IC community rather than DFINITY.
3. Create a social network of NNS neurons. 
4. Deploy 2 and 3 on the Internet Computer to move toward 100% on-chain governance.


## References

- [How to use dfx to interact with NNS canisters instead of nns app](https://github.com/flyq/blogs/blob/master/Dfinity/How%20to%20use%20dfx%20to%20interact%20with%20NNS%20canisters%20instead%20of%20nns%20app_en.md#how-to-use-dfx-to-interact-with-nns-canisters-instead-of-nns-app) by [Liquan](https://twitter.com/liquan_eth) of [Mix Labs](https://twitter.com/MixLabs_)
- [quill, minimalistic ledger and governance toolkit for cold wallets](https://github.com/dfinity/quill#:~:text=quill%20--pem-file%20%3Cpath%3E%20neuron-stake%20--amount%202.5%20--name%201,online%20machine%20using%20the%20send%20command%20from%20above), by [DFINITY](https://twitter.com/dfinity)
- [ICP staking with seed phrase and air-gapped computer](https://wiki.internetcomputer.org/wiki/ICP_staking_with_seed_phrase_and_air-gapped_computer) by [DFINITY](https://twitter.com/dfinity)
- [Introducing Quill, a Ledger and Governance Toolkit for the Internet Computer](https://medium.com/dfinity/introducing-quill-a-ledger-and-governance-toolkit-for-the-internet-computer-1df086ce5642) by [DFINITY](https://twitter.com/dfinity)
- [NNS API](https://raw.githubusercontent.com/dfinity/ic/master/rs/nns/governance/canister/governance.did) by [DFINITY](https://twitter.com/dfinity)
- [NNS API Governance Parameters](https://github.com/dfinity/ic/blob/master/rs/nns/governance/proto/ic_nns_governance/pb/v1/governance.proto) by [DFINITY](https://twitter.com/dfinity)

Herbert Yang, May 2022
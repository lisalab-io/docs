---
description: Rule based governance for LISA
---

# LISA DAO

## What is a DAO?

A DAO, or Decentralized Autonomous Organization, is a type of organization represented by rules encoded as a computer program that is transparent, and controlled by organization members. DAOs operate on blockchain technology, to automate and enforce rules and decisions without the need for a central authority.

## What is LISA DAO?

LISA DAO manages proposals to the LISA platform which enables changes to be implemented in a rule-based way.

Changes to the platform can take various forms, including modifications to parameters, minting or burning of tokens, and adding, upgrading, or deprecating system contracts.

## LISA DAO Architecture

<figure><img src="https://lh7-us.googleusercontent.com/docsz/AD_4nXfNSvCdi48Qd-5y3noOlqts7byDkfHWSwazCEPXuv-6ScMWK9lkJlcXgN2Azk0Lb093w3KCze_mPMf-ihd03si_rKcr3Qhwp8zXPV3F4BGY4zyn9V92fu3R1Vb9_trAggOWuFLbyng3C997aN9h5m3PXLM?key=BGAidPGZ_lZDLncVZe6njQ" alt=""><figcaption></figcaption></figure>

### Main DAO Components Overview

This architecture has the following main components:

#### LISA-DAO Contract

This is the core contract responsible for the DAO's operations and privileges. It is in charge of:

* Bootstrapping the DAO
* Defining new DAO features through extensions (modules)
* Executing proposals
* Authorizing components and extensions

#### Operators Contract

This contract manages the governance of the DAO and is implemented within LISA's platform as an extension. It is responsible for:

* Managing and controlling the users with permission to propose and vote changes to the platform.
* Establishing proposal signaling thresholds
* Receive and hold presented proposals while being  voted/accepted
* Signaling (voting on) registered proposals
* Delegating proposal execution to the DAO operation contract
* Controlling the validity, resubmissions, and expirations of proposals

#### Extension Contracts

These individual contracts are designed to enhance the features and operations of the DAO by providing new specific implementations. These contracts usually have the same permissions as the DAO contract itself. (see is-dao-or-extension)

#### Proposal Contracts

Proposals are contracts that define updates to the DAO platform. They are meant to execute tasks reserved for the DAO's privileged executors (the LISA DAO contract itself, or enabled extensions). These tasks can include platform state changes, executing actions, and introducing extensions to the platform. Each proposal is approved through a voting system implemented in the operator's extension. They are executed only once with the DAO contract as the transaction sender.

### Operations  Proposal **Lifecycle**

<figure><img src="https://lh7-us.googleusercontent.com/docsz/AD_4nXf-DqoRnlRuqEbYe_8AFiwkQd8FgmMyYLPRK7PJypjFsSGQdjFai7jt9qQ_-oCyNAJGZj9exHPDM0X6YV_NJRy8S9Jub5ofLvSedN1T0_ZwUr7ygnBLVQDF-q8Gx1mkpuuzvQTbFbzTKQiVj2xXnmPtJII?key=BGAidPGZ_lZDLncVZe6njQ" alt=""><figcaption></figcaption></figure>

**Proposal created**

A proposal is ready to be used when its contract is finally deployed.

**New Proposal**

Procedure to register the address of a given contract that conforms to the predefined proposal interface (trait).&#x20;

The operator that creates the proposal automatically signals 1 vote for it. Resubmissions (i.e. proposing again) are restricted only to non-executed expired proposals. This operation is restricted to enabled operators.

**Signaling**

Signaling is the process of voting for a certain registered and non-expired proposal in a boolean fashion (i.e., adding or subtracting 1 vote).&#x20;

Operators can only signal the same proposal twice if it has been resubmitted. The signaling procedure will trigger the execution step automatically if the proposal's signal count meets the threshold for execution. This operation is restricted to enabled operators.

**Execution**

As described in the signaling procedure, execution happens automatically when the signal count for the proposal meets the threshold.&#x20;

The procedure is delegated to the DAO operation contract, which restricts the operation to authorized extensions or the DAO contract itself. Finally, the execution invokes the proposal contract's execution method.

**Expiration**

Proposal expiration occurs at a certain point in time (in the underlying burn blockchain) when the block height surpasses the validity period for that proposal.

The validity period is defined as the range of blocks from when the proposal was created until reaching the sum of that block plus a predefined delta (currently set to 144 blocks).

**Validation Logic**

Proposals have a validation status, which is accomplished if two conditions are met:

* The proposal has not expired.
* The proposal was created after the operator extension was set up, including any updates to operators or thresholds.

This validation is used in both new proposals and signaling procedures.

#### Extensions

#### &#x20;**New extension**

Only DAO operator members (i.e. the core contract lisa-dao or previously registered and enabled extensions) can perform this step. Typically, new extensions are only registered through proposal contracts when they receive a majority of votes.\
\
**Availability**

During or after the registration of a module (extension), other DAO operator members can enable or disable the extension in a boolean fashion. Although disabled extensions will remain registered in the core contract, they will not be permitted to perform future operations while in that state.

As with the registration process, only DAO operator members have the authority to change the status of extensions.\


**Callbacks**

Extensions that conform to the predefined extension interface (trait) have access to the core contract’s "Extension Requests" special feature. This feature allows other enabled extensions to call the core contract, which in turn, can call back the initiating extension to perform a specific callback method with lisa-dao privileges. This includes the sender that triggered the call and a provided payload.

## Examples

### A proposal: lip-004.clar.

The lip-004 contract is a multi-purpose proposal that modifies the DAO-platform structure by adding and removing extensions while performing a few reconfigurations.

For it to be executed, it must first be accepted by the platform. The process is as follows:

* The newly proposed extension contracts (lqstx-mint-endpoint-v2-01 and public-pools-strategy-manager-v2) must be created along with this proposal contract, which will contain the code for the system reconfiguration.
* Once both contracts are created and deployed, the proposal is submitted to the DAO-platform through the operators (extension) contract. This contract verifies that the submission is made by a DAO-operator.
* The proposal is then put to a vote by the remaining DAO-operators (it is assumed that the submitting operator votes in favor, and its vote is automatically counted with the submission).
* At some point, the proposal either reaches a predefined voting threshold or it expires. If the voting threshold is reached, the operators contract invokes the executor-dao contract to run the proposal's execute() code with DAO privileges. In this way, the code in the proposal gains the necessary permissions, allowing it to invoke any required DAO-platform functionality as if it were already part of the DAO. This enables the proposal to do things like:
  * Include new extensions to the platform (as well as remove, stop, pause, etc.).
  * Reconfigure token pools.
  * Reconfigure blocklists.
  * Confirm special cases of token migrations.
  * Perform other necessary tasks.
* After all actions by the proposal are executed, it is discarded and cannot execute its code again or regain DAO privileges.

#### In this particular case

As mentioned before, in this particular case, the proposal contains code to perform different reconfiguration actions:

1. Newly deployed extension contracts must be set up in the DAO-platform, while the replaced versions must be removed from the platform.
2. Contracts that are removed from the platform must be also paused.
3. One of the new extensions requires initial setup.

To perform the extension replacement in the platform, a new invocation is made to the executor-dao contract, which tracks the extensions in use and allows verification of whether an invoking contract is approved.

The deprecated extensions must be paused to prevent them from being operated if invoked.

Finally, the public-pools-strategy-manager-v2 extension will be invoked to perform its initial setup.

DAO-operator: a contract already integrated into the DAO-platform or a system user with operator privileges.

<figure><img src="https://lh7-us.googleusercontent.com/docsz/AD_4nXfdABF0C-qTob-ANb244xMrVw6Jtd3W0eoynXlfbOUo6pINA0BVv42IZeaiDKGFmNslRDC_UAqQ66-ywtvZDjJOiohZg3uZE9vPqROzrIvz8Q6b6NmXfRs0cd0zFbpZTAsmwa2IiqjeMXHP9_jUWbUzzU8V?key=BGAidPGZ_lZDLncVZe6njQ" alt=""><figcaption></figcaption></figure>

### Extension Example: Treasury

Is a crucial part of the DAO’s financial management. It allows authorized transfers of both STX and SIP-010 tokens, ensuring that only the DAO or its extensions can initiate these transfers. Additionally, it provides functionality for making proxy calls to other contracts, further enhancing the DAO's capability to interact with and manage external contracts. This setup ensures secure and authorized management of the DAO’s treasury.

\

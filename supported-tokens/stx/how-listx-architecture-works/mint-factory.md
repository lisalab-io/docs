# Mint Factory

## LiSTX Mint Factory

LiSTX Mint Factory is the main smart contract LISA users interact to mint/burn LiSTX/vLiSTX.&#x20;

LiSTX Mint Factory implements the logic of LiSTX minting/burning process, while the relevant data is stored at LiSTX Mint Registry. Such separation of the logic and the data storage allows for future upgrades of the logic, while preserving the data layout.

### Request to mint LiSTX

When a user requests to mint LiSTX, a mint request ticket (in form of NFT) is issued to the user and the relevant STX is sent to LISA Vault.

In order for a user to start stacking from the next cycle, a mint request must be confirmed at least 300 blocks before the prepare stage of stacking begins for the next cycle (100 blocks before the current cycle end). Otherwise, the mint request is queued for the following cycle.

### Finalise the mint request

When LiSTX is ready to be minted (which is 432 blocks after the relevant stacking cycle start), the user can finalise the mint request at which time the mint request ticket will be burnt in exchange for LiSTX.

### Revoke the mint request

Subject to availability of unlocked STX at LISA Vault, a user may revoke a mint request, in which case the mint request ticket is burnt and the relevant STX sent to the user.

A user may not revoke a mint request if the unlocked STX at LISA Vault were sent to Strategies for stacking, or if the unlocked STX were redeemed to meet any outstanding burn requests.

### Request to burn LiSTX

When a user requests to burn LiSTX, a burn request ticket (in form of NFT) is issued to the user and the relevant LiSTX is sent to LISA Vault.&#x20;

While the user waits to burn LiSTX, the stacking rewards will continue to be accrued.

### Finalise the burn request

Finalisation of a burn request is subject to the availability of unlocked STX at LISA Vault. When a burn request is made, LiSTX Mint Factory will try to finalise immediately (in which case the user would immediately burn the burn request ticket and receive the relevant STX). Otherwise, the burn request is queued.

### Revoke the burn request

A user may revoke a burn request any time, in which case the burn request ticket will be burnt and the relevant LiSTX sent to the user.

## LiSTX and vLiSTX

LiSTX may be wrapped to vLiSTX any time, and vice versa.

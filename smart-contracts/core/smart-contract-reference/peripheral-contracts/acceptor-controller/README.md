# Acceptor Controller

## 1. Summary

IPWNAcceptorController.sol defines an interface each Acceptor Controller contract has to implement. Acceptor Controllers allow users to define specific eligibility parameters for their counterparties, such as World ID verification, OFAC compliance, and many others.

## 2. Important links

{% embed url="https://github.com/PWNDAO/pwn_protocol/blob/v1.4/src/interfaces/IPWNAcceptorController.sol" %}

## 3. Implementation

```solidity
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity 0.8.16;

import { IERC20 } from "MultiToken/MultiToken.sol";


/**
 * @title IPWNAcceptorController
 * @dev Interface for a contract that checks if a given proposal acceptor is valid.
 */
interface IPWNAcceptorController {

    /**
     * @notice Check if the proposal acceptor is valid.
     * @dev Must return IPWNAcceptorController interface id (0x75327ccb) or revert if the acceptor is not valid.
     * @param acceptor The address of the proposal acceptor.
     * @param proposerData The data of the proposal proposer.
     * @param acceptorData The data of the proposal acceptor.
     * @return The IPWNAcceptorController interface id (0x75327ccb) if the acceptor is valid.
     */
    function checkAcceptor(
        address acceptor, bytes memory proposerData, bytes memory acceptorData
    ) external view returns (bytes4);

}
```


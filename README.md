# upgradeable-contracts

3 ways to upgrade a smart contract:

1. parameterization Method

Simplest way of "upgrading our smart contract". Parameterizing everything, so that we can update the parameters at any point in time. Basically setter functions that change variables...
- Can't add new storage
- Can't add new logic

Example: 

```solidity
uint256 public reward;

function setReward(uint256 _reward) public onlyOwner{
  reward = _reward;
}
```

2. Social Migration Method

You just deploy the new contract, not connected in any way to the old contract, and you tell everybody by social convention to use the new contract.

Pros:
- Truest to Blockchain values
- Easiest to audit

Cons:
- Lot of work to convince users to move
- Different addresses

3. Proxies

Truest form of upgrades. It uses a lot of low level functionalities, the main one being the delegate call functionality.

## Delegate Call

Delegate call is a low level function where the code in the target contract is executed in the context of the calling contract. Msg.sender and msg.value are unchanged.

This combined with a fallback function can be used to delegate all calls through a proxy contract address to some other contract

Example:

```solidity
contract Proxy {
  fallback() external payable {
    address implementation = payable(0x3002...abc);
    implementation.delegatecall{msg.value}(msg.data);
}
```

Whenever we want to update the implementation, we just deploy a new implementation contract and point the proxy towards the new implementation.
All storage variables are stored in the proxy contract.

### Delegate call Details

When we're using delegateCall and updating the storage values of the proxy contract according to the implementation logic, we don't really care about the names of the state variables. We just need to make sure that they have the same types and that they fall under the same storage slot in both the proxy and implementation contract.


## Proxy Terminology
  1. The implementation contract:
       Has all the code of the protocol. To upgrade, we launch a new implementation contract
  2. The proxy contract:
       Points to which implementation is the correct one, and routes all function calls to that contract
  3. The user:
       make calls to the proxy
  4. The admin:
      User(group of users/voters) who upgrade to new implementation contracts

## Biggest Gotchas for Proxies:
  1. Storage Clashes
  2. Function selector Clashes


## Implementations Of Proxy Contracts

### 1. Transparent Proxy Pattern

Transparent Proxy Pattern includes the upgrade functionality in the proxy contract.

The proxy contract should have the "upgradeTo(address)" method, which will update the address of the implementation contract.

Admins can only call admin functions: functions in the proxy contract that govern the upgrade. Users can only call implementation functions. (Delegation of contract calls depends on the caller's address, delegated to implementation contract only if the caller is not a proxy admin).


### 2. Universal Upgradeable Proxies (UUPS)

This version of upgradeable contracts puts all the logic of upgrading in the implementation itself instead of the proxy.

Pros:
- Gas Saver
- Smaller Proxy
- Solidity will detect function selector clashes

Cons:
- If you forget to add upgrading functions in the implementation contract, you're stuck

### 3. Diamond Pattern

It allows for multiple implementation contracts. It also allows for smaller upgrades to be done.

### EIP-1867: Standard Proxy Storage Slots

Ethereum Improvement Proposal for having certain storage slots specifically used for proxies.

For example:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.19;

import "@openzeppelin/contracts/proxy/Proxy.sol";

contract SmallProxy is Proxy {
    // This is the keccak-256 hash of "eip1967.proxy.implementation" subtracted by 1
    bytes32 private constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;

    function setImplementation(address newImplementation) public {
        assembly {
            sstore(_IMPLEMENTATION_SLOT, newImplementation)
        }
    }
```

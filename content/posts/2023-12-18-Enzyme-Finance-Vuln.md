---
layout: post
title: 'Dive into the real world: 0x01'
description: This article is about vulnerability in the Enyzeme Finance. Lack of Trusted Forwarder Validation, Missing Gas Fee Validation, Relay Request Abuse Leading to Vault Exhaustion
date: 2023-12-18 02:00:53
author: anonymous
category: 'Bug Case'
---
## 개요

2023년 3월 28일에 한 연구원에 의해서 Enyzeme Finance에서 취약점이 제보되었다. 해당 취약점은 [`Gas Station Network(GSN)`](https://github.com/opengsn/gsn)라는 네트워크의 함수를 Enzyeme Finance에서 재정의하여 사용하는 과정에서 포워드 검증이 미흡하게 구현되어 발생했다.

---
## Enzyme Finance란

Enzyme Finance은 이더리움기반의 온체인 자산 관리 프로토콜이다. 이 프로토콜을 이용하면 사용자는 암호화폐 및 기타 디지털 자산과 같은 다양한 자산을 이용하여 맞춤형 투자 전략을 생성, 관리 및 투자 등의 서비스를 이용할 수 있다. 또한 이 Enzyme는 GSN를 이용하여 사용자가 Gas Fee를 지불하지 않고, Ethereum 스마트 계약과 상호 작용할 수 있도록 한다.

---
## Gas Station Network란

GSN는 DApp을 사용하는 사용자들 대신 수수료를 지불하는 분산형 중계기 네트워크이다. DApp을 사용하기 위해서는 사용자가 매 트랜잭션마다 수료를 지불해야 하며, 이전에 사용자는 Ethereum을 개인 지갑에 충전을 해야한다. 이러한 특성 때문에 DApp은 일반 사용자들에게는 진입 장벽이 높을 수가 있지만 이 GSN가 적용되어 있는 Enzyme에서는 사용자가 가스 수수료를 지불하지 않고도 사용할 수 있기 때문에 진입 장벽을 낮출 수 있다. 

GSN는 `Meta Transaction`을 사용하는데 이를 간단하게 설명하면 DApp을 사용할 때 최종 사용자가 가스 비용을 지불해야 할 필요성을 없애는 간단한 개념이다. 즉, 사용자는 자신도 모르게 블록체인을 사용하며, 외부 지갑을 연결하거나 거래소에서 Ethereum을 구매할 필요 없이 블록체인을 사용한다는 것이다. 

![](/img/blockchain/gns-process.jpg)

### Relay Server

중개 서버는 사용자의 트랜잭션을 받고, 해당 트랜잭션을 중개하고, 이를 블록체인으로 브로드 캐스팅하는 서버이이다. 이 중개 서버는 가스 비용을 내야하는 중개자의 Account Key를 제어한다.

### Paymaster

Paymaster는 GSN에서 중요한 역할을 하는 컨트랙트 중 하나이다. 메타 트랜잭션을 승인할지 거부할지를 결정하는 다양한 비즈니스 로직이 포함되며 애플리케이션은 논리를 사용자 정의하여 지불해야 할 거래와 거부해야 하는 거래를 정의할 수 있다. 이뿐만 아니라 Relay Server로 지불되는 가스 비용을 관리하고 조정한다.

### Relay Hub

RelayHub는 GSN에서 사용자, Relay Server, Paymaster간의 연결 지점을 관리하며 상호 작용하도록 도와주는 GSN에서 중요한 역할을 하는 컨트랙트 중 하나이다.

### Trusted Forwarder

Forwarded 컨트랙트는 GSN 네트워크 내에서 신뢰할 수 있는 유일한 구성 요소이며 Recipient 컨트랙트는 원래 발신자의 메시지와 nonce를 확인하기 위해 Forwarded를 사용한다.

### Meta Transaction 흐름
![](/img/blockchain/meta-transaction.jpg)

1. Transaction의 세부 정보가 포함된 서명된 메시지를 중개 Relay Server로 전송한다.
2. Relay Server는 Transaction을 확인하고, 비용을 충당할 만큼 충분한 수수료가 있는지 확인한다.
3. Relay Server는 사용자의 서명된 메시지, 신뢰할 수 있는 Forwarder의 주소, Paymaster의 주소를 사용하여 Relay Hub를 호출하는 새로운 Transaction을 생성한다.
4. Relay Server는 새로운 Transaction을 서명하고, 이더리움 네트워크로 전송하며, 필요한 가스 비용을 미리 지불한다.
5. 이 Transaction을 받는다면 Relay Hub는 사용자의 서명된 메시지와 함께 신뢰하는 Forwarder 컨트랙트를 호출하고, Recipient 컨트랙트를 호출한다.
6. 신뢰하는 Forwarder 컨트랙트는 사용자의 시그니처를 검증하고, 사용자의 주소를 복구하고 Transaction을  Recipient 컨트랙트로 전송한다.
7. Transaction이 실행되고, Recipient 컨트랙트에 의해서 블록체인의 상태가 업데이트 된다.
8. Transaction이 완료된 후, Relay Server는 가스비를 지불했으므로 이를 Relay Hub에게 보고한다. Relay Server는 Relay Hub에게 가스비에 대한 보상을 요청한다.
9. Paymaster 컨트랙트는 거래를 검증하고 가스 요금 및 추가 서비스 요금을 충당하기 위해 자금(토큰 또는 ETH)을 Relay Server로 보낸다.

Meta Transaction의 흐름 과정은 위와 같이 정리할 수 있다.

### 주요 컨트랙트 분석
#### StakeManager
```
/// Only the owner can call this function. If the entry does not exist, reverts.
/// @param relayManager - address that represents a stake entry and controls relay registrations on relay hubs
/// @param unstakeDelay - number of blocks to elapse before the owner can retrieve the stake after calling 'unlock'
function stakeForRelayManager(address relayManager, uint256 unstakeDelay) external payable;
(skip)
function withdrawStake(address relayManager) external;
(skip)
function isRelayManagerStaked(address relayManager, address relayHub, uint256 minAmount, uint256 minUnstakeDelay)
    external
    view
    returns (bool);
(skip)
/// Slash the stake of the relay relayManager. In order to prevent stake kidnapping, burns half of stake on the way.
/// @param relayManager - entry to penalize
/// @param beneficiary - address that receives half of the penalty amount
/// @param amount - amount to withdraw from stake
function penalizeRelayManager(address relayManager, address payable beneficiary, uint256 amount) external;
```
RelayHub는 위에서 설명한 것처럼 사용자, Relay Server, Paymaster간의 연결 지점을 관리하며 상호 작용하도록 도와주는 GSN에서 중요한 역할을 하는 컨트랙트라고 했다. 이 Relay Hub는 등록 관리를 담당하는 중계 관리자라는 개체를 등록하여 운영된다. 중개자는 특정 중개 관리자에게 할당되며 그 관리자를 통해서 Relay Request를 제출한다.

위 함수들은 `IStakeManager.sol`라는 인터페이스 파일 내에 정의되어 있다. `stakeForRelayManager()` 함수는 중개 관리자가 ERC 20 토큰을 스테이킹할 수 있는 함수이며, `withdrawStake()` 함수는 중개 관리자가 지분을 철회할 수 있는 함수이며, `isRelayManagerStaked()` 함수는 Stake 관리자가 유효한지 확인하기 위해서 RelayHub에서 호출되는 주요 함수이다. 그리고 중개 작업자가 잘못된 행동을 할 경우에는 해당 중개 작업자와 연관된 중개 관리자를 처벌할 수 있는데 이때 사용되는 함수이다.

#### Forwarder
```
 		/**
     * verify the transaction would execute.
     * validate the signature and the nonce of the request.
     * revert if either signature or nonce are incorrect.
     * also revert if domainSeparator or requestTypeHash are not registered.
     */
    function verify(
        ForwardRequest calldata forwardRequest,
        bytes32 domainSeparator,
        bytes32 requestTypeHash,
        bytes calldata suffixData,
        bytes calldata signature
    ) external view;

    /**
     * execute a transaction
     * @param forwardRequest - all transaction parameters
     * @param domainSeparator - domain used when signing this request
     * @param requestTypeHash - request type used when signing this request.
     * @param suffixData - the extension data used when signing this request.
     * @param signature - signature to validate.
     *
     * the transaction is verified, and then executed.
     * the success and ret of "call" are returned.
     * This method would revert only verification errors. target errors
     * are reported using the returned "success" and ret string
     */
    function execute(
        ForwardRequest calldata forwardRequest,
        bytes32 domainSeparator,
        bytes32 requestTypeHash,
        bytes calldata suffixData,
        bytes calldata signature
    )
```
Forwarder 컨트랙트는 2가지의 주요 기능만 가진 간단한 컨트랙트이다. `verify()` 함수는 보낸 사람이 메시지 데이터에 올바르게 서명했는지 확인하는 함수이고, `execute()` 함수는 수신자 트랜잭션에서 요청 데이터의 내용을 실행하는 함수이다.

```
function verify(
    ForwardRequest calldata req,
    bytes32 domainSeparator,
    bytes32 requestTypeHash,
    bytes calldata suffixData,
    bytes calldata sig)
external override view {

    _verifyNonce(req);
    _verifySig(req, domainSeparator, requestTypeHash, suffixData, sig);
}
```
구현되어 있는 `verify()` 함수를 보면 `_verifyNonce()` 함수를 이용해서 nonce를 확인하고 `_verifySig()` 함수를 이용해서 보낸 사람의 서명을 확인하는 것을 볼 수 있다. 그리고 verify는 EIP-712를 사용하고 있다.

```
function _verifyNonce(ForwardRequest calldata req) internal view {
    require(nonces[req.from] == req.nonce, "FWD: nonce mismatch");
}

function _verifySig(
    ForwardRequest calldata req,
    bytes32 domainSeparator,
    bytes32 requestTypeHash,
    bytes calldata suffixData,
    bytes calldata sig)
internal
view {
    require(domains[domainSeparator], "FWD: unregistered domain sep.");
    require(typeHashes[requestTypeHash], "FWD: unregistered typehash");
    bytes32 digest = keccak256(abi.encodePacked(
        "\x19\x01", domainSeparator,
        keccak256(_getEncoded(req, requestTypeHash, suffixData))
    ));
    require(digest.recover(sig) == req.from, "FWD: signature mismatch");
}
```
_verifyNonce() 함수와 _verifySig() 함수는 위와 같이 구현되어 있다. 

```
function execute(
    ForwardRequest calldata req,
    bytes32 domainSeparator,
    bytes32 requestTypeHash,
    bytes calldata suffixData,
    bytes calldata sig
)
external payable
override
returns(bool success, bytes memory ret) {
    _verifySig(req, domainSeparator, requestTypeHash, suffixData, sig);
    _verifyAndUpdateNonce(req);

    require(req.validUntil == 0 || req.validUntil > block.number, "FWD: request expired");

    uint gasForTransfer = 0;
    if (req.value != 0) {
        gasForTransfer = 40000; //buffer in case we need to move eth after the transaction.
    }
    bytes memory callData = abi.encodePacked(req.data, req.from);
    require(gasleft() * 63 / 64 >= req.gas + gasForTransfer, "FWD: insufficient gas");
    // solhint-disable-next-line avoid-low-level-calls
    (success, ret) = req.to.call {
        gas: req.gas,
        value: req.value
    }(callData);
    if (req.value != 0 && address(this).balance > 0) {
        // can't fail: req.from signed (off-chain) the request, so it must be an EOA...
        payable(req.from).transfer(address(this).balance);
    }

    return (success, ret);
}
```
`execute()` 함수는 위와 같이 구현되어 있다. Recipient 컨트랙트를 호출해서 메시지 내용을 실행한다. 이때 Recipient 컨트랙트는 request.to에 저장되고, 이 컨트랙트는 Forwarder를 신뢰하고 모든 기능을 호출할 수 있도록 허용한다.

#### Recipient
```
// SPDX-License-Identifier: MIT
// solhint-disable no-inline-assembly
pragma solidity >=0.6.9;

import "./interfaces/IRelayRecipient.sol";

/**
 * A base contract to be inherited by any contract that want to receive relayed transactions
 * A subclass must use "_msgSender()" instead of "msg.sender"
 */
abstract contract BaseRelayRecipient is IRelayRecipient {

    /*
     * Forwarder singleton we accept calls from
     */
    address private _trustedForwarder;

    function trustedForwarder() public virtual view returns (address){
        return _trustedForwarder;
    }

    function _setTrustedForwarder(address _forwarder) internal {
        _trustedForwarder = _forwarder;
    }

    function isTrustedForwarder(address forwarder) public virtual override view returns(bool) {
        return forwarder == _trustedForwarder;
    }

    /**
     * return the sender of this call.
     * if the call came through our trusted forwarder, return the original sender.
     * otherwise, return `msg.sender`.
     * should be used in the contract anywhere instead of msg.sender
     */
    function _msgSender() internal override virtual view returns (address ret) {
        if (msg.data.length >= 20 && isTrustedForwarder(msg.sender)) {
            // At this point we know that the sender is a trusted forwarder,
            // so we trust that the last bytes of msg.data are the verified sender address.
            // extract sender address from the end of msg.data
            assembly {
                ret := shr(96,calldataload(sub(calldatasize(),20)))
            }
        } else {
            ret = msg.sender;
        }
    }

    /**
     * return the msg.data of this call.
     * if the call came through our trusted forwarder, then the real sender was appended as the last 20 bytes
     * of the msg.data - so this method will strip those 20 bytes off.
     * otherwise (if the call was made directly and not through the forwarder), return `msg.data`
     * should be used in the contract instead of msg.data, where this difference matters.
     */
    function _msgData() internal override virtual view returns (bytes calldata ret) {
        if (msg.data.length >= 20 && isTrustedForwarder(msg.sender)) {
            return msg.data[0:msg.data.length-20];
        } else {
            return msg.data;
        }
    }
}
```
중재된 Transaction을 받고 싶은 모든 컨트랙트는 BaseRelayRecipient 컨트랙트를 구현해야한다. `trustedForwarder()`,  `_setTrustedForwarder()`, `isTrustedForwarder()` 함수를 이용해서 신뢰하는 Forwarder를 설정하고, Forwarder가 신뢰할 수 있는지 확인하는 함수이다. `_msgSender()`, `_msgData()` 함수는 msg.sender이 신뢰할 수 있는 Forwarder일 경우 original sender/data를 가져온다.

### Paymaster
```
function preRelayedCall(
    GsnTypes.RelayRequest calldata relayRequest,
    bytes calldata signature,
    bytes calldata approvalData,
    uint256 maxPossibleGas
)
external
returns(bytes memory context, bool rejectOnRecipientRevert);

function postRelayedCall(
    bytes calldata context,
    bool success,
    uint256 gasUseWithoutPost,
    GsnTypes.RelayData calldata relayData
) external;
```
IPaymaster 인터페이스를 사용하면 Paymaster 로직을 사용자 정의하여 중개자의 요청을 처리할 수 있다. 이때 중요한 두 가지의 기능은 `preRelayedCall()`, `postRelayedCall()` 함수이다.

`preRelayedCall()` 함수는 Paymaster가 중계된 호출(relayed call)을 지불할 것인지 여부를 확인하는 함수로 RelayHub에서 항상 호출된다. 그리고 `preRelayedCal()` 함수가 Revert를 호출하지 않으면 중계된 호출의 가스 비용을 부담한다.

`postRelayedCall()` 함수는 Recipient 계약에서 요청된 호출(request call)이 실행된 후에 RelayHub에서 호출되는 함수이다. 이 기능 트랜잭션 기록, 이벤트 발생, Paymaster 잔액 보충과 같은 작업을 수행하며 Enzyme Finance 또한 이를 Paymaster 잔액 보충으로 사용하고 있다.

```
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity ^0.8.0;
pragma abicoder v2;

import "../forwarder/IForwarder.sol";
import "../BasePaymaster.sol";

contract TestPaymasterEverythingAccepted is BasePaymaster {

    function versionPaymaster() external view override virtual returns (string memory){
        return "2.2.3+opengsn.test-pea.ipaymaster";
    }

    event SampleRecipientPreCall();
    event SampleRecipientPostCall(bool success, uint actualCharge);

    function preRelayedCall(
        GsnTypes.RelayRequest calldata relayRequest,
        bytes calldata signature,
        bytes calldata approvalData,
        uint256 maxPossibleGas
    )
    external
    override
    virtual
    returns (bytes memory, bool) {
        (signature);
        _verifyForwarder(relayRequest);
        (approvalData, maxPossibleGas);
        emit SampleRecipientPreCall();
        return ("no revert here",false);
    }

    function postRelayedCall(
        bytes calldata context,
        bool success,
        uint256 gasUseWithoutPost,
        GsnTypes.RelayData calldata relayData
    )
    external
    override
    virtual
    {
        (context, gasUseWithoutPost, relayData);
        emit SampleRecipientPostCall(success, gasUseWithoutPost);
    }

    function deposit() public payable {
        require(address(relayHub) != address(0), "relay hub address not set");
        relayHub.depositFor{value:msg.value}(address(this));
    }

    function withdrawAll(address payable destination) public {
        uint256 amount = relayHub.balanceOf(address(this));
        withdrawRelayHubDepositTo(amount, destination);
    }
}
```
위 컨트랙트 코드는 GSN 네트워크에서 제공하는 Paymaster 예시 컨트랙트 코드이다. 위 코드에서 기억해야 할 부분은 `preRelayedCall()`, `postRelayedCall()` 함수가 구현되어 있고, preRelayedCall() 함수 내에서 Forwarder를 검증하는 `_verifyForwarder()` 함수가 구현되어 있다는 것이다. `postRelayedCall()` 함수의 context는 `preRelayedCall()` 함수의 output에 의해 제공되고, gasUseWithoutPost는 현재까지 사용된 정확한 가스이다. 

```
function _verifyForwarder(GsnTypes.RelayRequest calldata relayRequest)
public
view {
    require(address(_trustedForwarder) == relayRequest.relayData.forwarder, "Forwarder is not trusted");
    GsnEip712Library.verifyForwarderTrusted(relayRequest);
}

// other lib
function verifyForwarderTrusted(GsnTypes.RelayRequest calldata relayRequest) internal view {
    (bool success, bytes memory ret) = relayRequest.request.to.staticcall(
        abi.encodeWithSelector(
            IRelayRecipient.isTrustedForwarder.selector, relayRequest.relayData.forwarder
        )
    );
    require(success, "isTrustedForwarder: reverted");
    require(ret.length == 32, "isTrustedForwarder: bad response");
    require(abi.decode(ret, (bool)), "invalid forwarder for recipient");
}
```
`_verifyForwarder()` 함수에서 첫 번째 검증에서는 요청 데이터 내에 포함되어 있는 forwarder가 신뢰하는 forwarder 주소인지 확인하고, 두 번째 검증에서는 Relay request의 forwarder가 recipient 컨트랙트에 저장되어 있는 신뢰된forwarder와 일치하는지 확인한다.

### RelayHub
```
/// Relays a transaction. For this to succeed, multiple conditions must be met:
///  - Paymaster's "preRelayCall" method must succeed and not revert
///  - the sender must be a registered Relay Worker that the user signed
///  - the transaction's gas price must be equal or larger than the one that was signed by the sender
///  - the transaction must have enough gas to run all internal transactions if they use all gas available to them
///  - the Paymaster must have enough balance to pay the Relay Worker for the scenario when all gas is spent
///
/// If all conditions are met, the call will be relayed and the recipient charged.
///
/// Arguments:
/// @param maxAcceptanceBudget - max valid value for paymaster.getGasLimits().acceptanceBudget
/// @param relayRequest - all details of the requested relayed call
/// @param signature - client's EIP-712 signature over the relayRequest struct
/// @param approvalData: dapp-specific data forwarded to preRelayedCall.
///        This value is *not* verified by the Hub. For example, it can be used to pass a signature to the Paymaster
/// @param externalGasLimit - the value passed as gasLimit to the transaction.
///
/// Emits a TransactionRelayed event.
function relayCall(
    uint maxAcceptanceBudget,
    GsnTypes.RelayRequest calldata relayRequest,
    bytes calldata signature,
    bytes calldata approvalData,
    uint externalGasLimit
)
external
returns(bool paymasterAccepted, bytes memory returnValue);
```
RelayHub는 GSN에서 사용자, Relay Server, Paymaster간의 연결 지점을 서로 상호 작용하도록 도와주는 게이트 웨이 역할을 한다고 설명했다. 먼저 중개자가 요청을 실행하기 위해서 `RelayCall()` 함수를 호출한다. 이 Relay 요청이 성공적으로 마치기 위해서는 IRelayHub.sol에 정의되어 있는 조건들을 모두 만족 시켜야 한다. 

1. preRelayedCall() 함수는 무조건 성공해야 하며 되돌릴 수 없다.
2. sender는 StakeManager에 적절한 양의 ERC 20 토큰을 스테이킹한 중개 관리자와 연결된 등록된 중개원이여야 한다.
3. Transaction의 gas price는 보낸 사람이 서명한 gas price와 같거나 커야 한다.
4. Transaction 내에는 모든 internal transaction을 수용할 수 있을 만큼 충분한 가스가 있어야 한다.
5. Paymaster는 중개자에게 서비스에 대한 보상을 제공할 수 있을 만큼 충분한 잔액을 보유하고 있어야 한다. 

위와 같은 조건들을 만족 시킨다면 Relay 요청은 성공적으로 마칠 수 있을 것이다.

```
uint maxAcceptanceBudget : 
GsnTypes.RelayRequest calldata relayRequest : 
bytes calldata signature : 중개자가 서명한 Relay 요청
bytes calldata approvalData : 사용자 정의 승인 로직을 위해 PreRelayesCall()로 전송된 데이터
uint externalGasLimit : 거래에 소비될 수 있는 최대 가스
```
이 함수에서 중요한 매개변수 중 하나는 maxAcceptanceBudget이다. 이는 preRelayedCall() 함수에서 사용되는 가스가 이 한도를 초과하지 않으면 Paymaster에 요금이 청구되지 않는다.

```
// GSNTypes.sol
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity ^0.8.0;

import "../forwarder/IForwarder.sol";

interface GsnTypes {
    /// @notice gasPrice, pctRelayFee and baseRelayFee must be validated inside of the paymaster's preRelayedCall in order not to overpay
    struct RelayData {
        uint256 gasPrice;
        uint256 pctRelayFee;
        uint256 baseRelayFee;
        address relayWorker;
        address paymaster;
        address forwarder;
        bytes paymasterData;
        uint256 clientId;
    }

    //note: must start with the ForwardRequest to be an extension of the generic forwarder
    struct RelayRequest {
        IForwarder.ForwardRequest request;
        RelayData relayData;
    }
}

// IForwarder.sol
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity >=0.7.6;
pragma abicoder v2;

interface IForwarder {

    struct ForwardRequest {
        address from;
        address to;
        uint256 value;
        uint256 gas;
        uint256 nonce;
        bytes data;
        uint256 validUntil;
    }
```
maxAcceptanceBudget에 이어서 가장 중요한 매개변수는 GsnTypes.RelayRequest 구조체이다. 이 구조체는 GSNTypes.sol 인터페이스에 포함되어 있다. 위 코드를 보면 RelayRequest 구조체가 정의되어 있고, IForwarder 인터페이스에는 ForwardRequest가 정의되어 있다.

- Gsntypes
    - gasPrice : 지불할 가스의 단위당 가격
    - pctRelayFee : 중계자에게 수수료로 지불할 비용의 비율
    - baseRelayFee : 지불과 상관없이 부과되는 기본료
    - paymaster : Paymaster는 Relay 요청을 수락하거나 거부
    - forwarder : 원래 보낸 사람이 보낸 메시지를 확인하는 신뢰할 수 있는 forwarder
    - paymasterData : true라면 Paymaster 잔액을 다시 0.5 이더로 충전
- IForwarder
    - from : Relay 요청을 하는 Vault 소유자
    - to : Receipient 컨트랙트 ( 이 경우 comptrollerLib 또는 VaultProxy)
    - value: 함수 호출에 전송될 이더리움 금액
    - gas : 이 거래를 실행하기 위해 전송된 가스
    - nonce : Replay 공격을 방지하기 위한 임시 메시지
    - data : 기능 선택기 및 입력 데이터 포함
    - validUntil : 마감 시간

각 구조체의 input의 목적은 위와 같다.

---
## GSN with Enzyme Finance

1. GasRelayPaymasterLib — Paymaster Contract
2. ComptrollerLib — Recipient Contract

Enzyme Finance에서는 Paymaster 컨트랙트는 GasRelayPaymasterLib, Recipient 컨트랙트는 ComptrollerLib라는 컨트랙트를 구현해서 사용하고 있었다.

### ComptrollerLib (Recipient)

ComptrollerLib는 Vault 소유자가 수행하는 다양한 기능을 처리하기 위해 Enzyme Finance에서 개발한 Recipient 컨트랙트이다. 업그레이드할 수 없는 프록시인 ComptrollerProxy를 ComptrollerLib에 위임하여 이를 통해서 ComptrollerLib로 엑세스할 수 있다.

```
/// @notice Calls a specified action on an Extension
/// @param _extension The Extension contract to call (e.g., FeeManager)
/// @param _actionId An ID representing the action to take on the extension (see extension)
/// @param _callArgs The encoded data for the call
/// @dev Used to route arbitrary calls, so that msg.sender is the ComptrollerProxy
/// (for access control). Uses a mutex of sorts that allows "permissioned vault actions"
/// during calls originating from this function.
function callOnExtension(address _extension, uint256 _actionId, bytes calldata _callArgs)
external
override
locksReentrance
allowsPermissionedVaultAction {
    require(
        _extension == getFeeManager() || _extension == getIntegrationManager() ||
        _extension == getExternalPositionManager(),
        "callOnExtension: _extension invalid"
    );

    IExtension(_extension).receiveCallFromComptroller(__msgSender(), _actionId, _callArgs);
}
```
호출할 주요 함수 중 하나는 `callOnExtenstion()`이라는 함수이다. 이 함수의 매개변수로는 확장자 계약의 주소, 해당 확장 계약에서 원하는 작업을 지정하는 작업 ID, callArgs이 있다.

```
/// @notice Makes an arbitrary call with the VaultProxy contract as the sender
/// @param _contract The contract to call
/// @param _selector The selector to call
/// @param _encodedArgs The encoded arguments for the call
/// @return returnData_ The data returned by the call
function vaultCallOnContract(address _contract, bytes4 _selector, bytes calldata _encodedArgs)
external
override
onlyOwner
returns(bytes memory returnData_) {
    require(
        IFundDeployer(getFundDeployer()).isAllowedVaultCall(_contract, _selector, keccak256(_encodedArgs)),
        "vaultCallOnContract: Not allowed"
    );

    return IVault(getVaultProxy()).callOnContract(_contract, abi.encodePacked(_selector, _encodedArgs));
}

/// @notice Buys back shares collected as protocol fee at a discounted shares price, using MLN
/// @param _sharesAmount The amount of shares to buy back
function buyBackProtocolFeeShares(uint256 _sharesAmount) external override {
    address vaultProxyCopy = vaultProxy;
    require(IVault(vaultProxyCopy).canManageAssets(__msgSender()), "buyBackProtocolFeeShares: Unauthorized");

    uint256 gav = calcGav();

    IVault(vaultProxyCopy).buyBackProtocolFeeShares(
        _sharesAmount, __getBuybackValueInMln(vaultProxyCopy, _sharesAmount, gav), gav
    );
}

/// @notice Sets whether to attempt to buyback protocol fee shares immediately when collected
/// @param _nextAutoProtocolFeeSharesBuyback True if protocol fee shares should be attempted
/// to be bought back immediately when collected
function setAutoProtocolFeeSharesBuyback(bool _nextAutoProtocolFeeSharesBuyback) external override onlyOwner {
    autoProtocolFeeSharesBuyback = _nextAutoProtocolFeeSharesBuyback;

    emit AutoProtocolFeeSharesBuybackSet(_nextAutoProtocolFeeSharesBuyback);
}
```
Vault는 확장 계약으로 작동할 수 있는 feeManager, IntegrationManager, externalPositionManager 컨트랙트를 인식한다. 또한 이 컨트랙트 내의 다른 대상 함수인 `vaultCallOnContract()`, `buyBackProtocolFeeShares()`, `setAutoProtocolFeeSharesBuyBack()`도 Relayer가 호출할 수 있다. 이 모든 함수는 볼트 소유자가 볼트 내의 자산을 관리할 때 수행하는 작업과 관련된 함수들이다.

### GasRelayPaymasterLib (Paymaster)

이 컨트랙트는 IGasRelayPayMaster에서 상속되며 위에서 설명한 `preRelayedCall()`, `postRelayedCall()`이라는 함수를 포함하고 있다. 이는 GasRelayPaymentFactory에 의해 배포되는 BeaconProxy이라는 프록시를 통해 액세스할 수 있다.

```
/// @notice Checks whether the paymaster will pay for a given relayed tx
/// @param _relayRequest The full relay request structure
/// @return context_ The tx signer and the fn sig, encoded so that it can be passed to `postRelayCall`
/// @return rejectOnRecipientRevert_ Always false
function preRelayedCall(
    IGsnTypes.RelayRequest calldata _relayRequest,
    bytes calldata,
    bytes calldata,
    uint256
)
external
override
relayHubOnly
returns(bytes memory context_, bool rejectOnRecipientRevert_) {
    address vaultProxy = getParentVault();
    require(
        IVault(vaultProxy).canRelayCalls(_relayRequest.request.from),
        "preRelayedCall: Unauthorized caller"
    );

    bytes4 selector = __parseTxDataFunctionSelector(_relayRequest.request.data);
    require(
        __isAllowedCall(
            vaultProxy,
            _relayRequest.request.to,
            selector,
            _relayRequest.request.data
        ),
        "preRelayedCall: Function call not permitted"
    );

    return (abi.encode(_relayRequest.request.from, selector), false);
}
```
GasRelayPaymasterLib.sol에 구현되어 있는 `preRelayedCall()` 함수를 보면 RelayHub만 이를 호출할 수 있는 것을 볼 수 있다. `IVault(vaultProxy).canRelayCalls()` 함수는 요청 발신자가 볼트 소유자인지 또는 Relay 호출을 할 수 있는 승인된 주소인지 확인한다. 이후에 `__isAllowedCall()` 함수를 호출하여 relay 요청의 기능 선택기가 사전 승인된 선택기 세트와 일치하는지 확인한다.

```
/// @dev Helper to check if a contract call is allowed to be relayed using this paymaster
/// Allowed contracts are:
/// - VaultProxy
/// - ComptrollerProxy
/// - PolicyManager
/// - FundDeployer
function __isAllowedCall(
    address _vaultProxy,
    address _contract,
    bytes4 _selector,
    bytes calldata _txData
) private view returns(bool allowed_) {
    if (_contract == _vaultProxy) {
        // All calls to the VaultProxy are allowed
        return true;
    }

    address parentComptroller = __getComptrollerForVault(_vaultProxy);
    if (_contract == parentComptroller) {
        if (
            _selector == ComptrollerLib.callOnExtension.selector ||
            _selector == ComptrollerLib.vaultCallOnContract.selector ||
            _selector == ComptrollerLib.buyBackProtocolFeeShares.selector ||
            _selector == ComptrollerLib.depositToGasRelayPaymaster.selector ||
            _selector == ComptrollerLib.setAutoProtocolFeeSharesBuyback.selector
        ) {
            return true;
        }
    } else if (_contract == ComptrollerLib(parentComptroller).getPolicyManager()) {
        if (
            _selector == PolicyManager.updatePolicySettingsForFund.selector ||
            _selector == PolicyManager.enablePolicyForFund.selector ||
            _selector == PolicyManager.disablePolicyForFund.selector
        ) {
            return __parseTxDataFirstParameterAsAddress(_txData) == getParentComptroller();
        }
    } else if (_contract == ComptrollerLib(parentComptroller).getFundDeployer()) {
        if (
            _selector == FundDeployer.createReconfigurationRequest.selector ||
            _selector == FundDeployer.executeReconfiguration.selector ||
            _selector == FundDeployer.cancelReconfiguration.selector
        ) {
            return __parseTxDataFirstParameterAsAddress(_txData) == getParentVault();
        }
    }

    return false;
}
```
`__isAllowedCall()` 함수는 위와 같다.

```
/// @notice Called by the relay hub after the relayed tx is executed, tops up deposit if flag passed through paymasterdata is true
/// @param _context The context constructed by preRelayedCall (used to pass data from pre to post relayed call)
/// @param _success Whether or not the relayed tx succeed
/// @param _relayData The relay params of the request. can be used by relayHub.calculateCharge()
function postRelayedCall(
    bytes calldata _context,
    bool _success,
    uint256,
    IGsnTypes.RelayData calldata _relayData
) external override relayHubOnly {
    bool shouldTopUpDeposit = abi.decode(_relayData.paymasterData, (bool));
    if (shouldTopUpDeposit) {
        __depositMax();
    }

    (address spender, bytes4 selector) = abi.decode(_context, (address, bytes4));
    emit TransactionRelayed(spender, selector, _success);
}
```
`postRelayedCall()` 함수는 위와 같다. 이 함수 또한 relayHubOnly 수정자를 통해서 RelayHub에서만 호출될 수 있도록 하는 것을 볼 수 있다. 또한 relay 요청으로 전달된 paymasterData의 값이 true라면 `__depositMax()` 함수를 호출하는 것을 볼 수 있다.

```
uint256 private constant DEPOSIT = 0.5 ether;
(skip)
// PRIVATE FUNCTIONS

/// @dev Helper to pull WETH from the associated vault to top up to the max ETH deposit in the relay hub
function __depositMax() private {
    uint256 prevDeposit = getRelayHubDeposit();

    if (prevDeposit < DEPOSIT) {
        uint256 amount = DEPOSIT.sub(prevDeposit);

        IGasRelayPaymasterDepositor(getParentComptroller()).pullWethForGasRelayer(amount);

        IWETH(getWethToken()).withdraw(amount);

        IGsnRelayHub(getHubAddr()).depositFor {
            value: amount
        }(address(this));

        emit Deposited(amount);
    }
}
```
`__depositMax()` 함수는 Paymaster에 0.5 ether를 충전한다. 이 잔액을 충전하기 위해 WETH를 Vault에서 가져와 eth로 변환한 다음 RelayHub로 전송한다.

![](/img/blockchain/asdf.jpg)

최종적으로 Enzyme Finance에서 Gas Relay의 흐름도는 위와 같다.

---
## 취약점 분석

사실 이미 위의 내용에서 간접적으로 취약점을 설명했다.

```
function _verifyForwarder(GsnTypes.RelayRequest calldata relayRequest)
public
view {
    require(address(_trustedForwarder) == relayRequest.relayData.forwarder, "Forwarder is not trusted");
    GsnEip712Library.verifyForwarderTrusted(relayRequest);
}

// other lib
function verifyForwarderTrusted(GsnTypes.RelayRequest calldata relayRequest) internal view {
    (bool success, bytes memory ret) = relayRequest.request.to.staticcall(
        abi.encodeWithSelector(
            IRelayRecipient.isTrustedForwarder.selector, relayRequest.relayData.forwarder
        )
    );
    require(success, "isTrustedForwarder: reverted");
    require(ret.length == 32, "isTrustedForwarder: bad response");
    require(abi.decode(ret, (bool)), "invalid forwarder for recipient");
}
```
GSN에서 제공해주는 Paymaster 테스트 컨트랙트인 TestPaymasterEverythingAccepted의 `preRelayedCall()` 함수에서는 `_verifyForwarder()` 함수를 호출해서 Paymaster와 recipient 모두가 forwarder를 승인하는지 확인하는 과정을 거친다.

![](/img/blockchain/asdf1.jpg)

오른쪽이 테스트 컨트랙트인 TestPaymasterEverythingAccepted이고, 왼쪽이 Enzyme Finance의 실제로 사용된 Paymaster 컨트랙트인 GasRelayPaymasterLib이다. GasRelayPaymasterLib의 `preRelayedCall()` 함수 내부를 보면 forwarder를 검증하는 함수가 없는 것을 확인할 수 있다. 이것이 첫 번째 취약점이다.

Enzyme 팀은 `preRelayedCall()`의 사용자 정의된 함수를 만들었지만 Paymaster와 recipient가 사용하는 forwarder가 Relay 요청에서 전송된 forwarder와 일치한지 검증하는 필수적인 검사를 빼먹은 것이다. 이 forwarder 컨트랙트의 유일한 역할은 원래 Relay 요청이 올바른 저장소 소유자에 의해 서명되었는지 확인하는 것이다. 이로 인해서 Recipient 컨트랙트는 forwarder를 신뢰하는 forwarder가 확인 작업을 이미 수행했다는 가정하고, 추가 확인 없이 메시지를 실행 시키는 것이다. 그럼 계속 설명했던 “신뢰”하는의 의미가 정확이 무엇인지 GSN의 RelayHub.sol 코드를 통해 이해를 해야 한다.

```
// Calls to the recipient are performed atomically inside an inner transaction which may revert in case of
// errors in the recipient. In either case (revert or regular execution) the return data encodes the
// RelayCallStatus value.
(bool success, bytes memory relayCallStatus) = address(this).call {
    gas: innerGasLimit
}(
    abi.encodeWithSelector(RelayHub.innerRelayCall.selector, relayRequest, signature, approvalData, vars.gasAndDataLimits,
        _tmpInitialGas - gasleft(),
        vars.maxPossibleGas
    )
);
```
1. preRelayedCall() 함수를 호출해 Paymaster가 요청을 수락하는지 확인
2. Relay 요청을 검증하기 위해 forwarder를 호출
3. Recipient에서 메시지 실행
4. postRelayedCall() 함수가 호출되어 필요한 경우 paymaster eth 잔액을 0.5 ether로 충전

RelayHub.sol의 RelayCall 함수의 snippet이다. 위 snippet을 보면 innerRelayCall() 함수를 호출하는데 이 함수는 총 위와 같은 4가지의 주요 작업이 수행된다.

```
// We now perform the actual charge calculation, based on the measured gas used
uint256 gasUsed = (externalGasLimit - gasleft()) + config.gasOverhead;
uint256 charge = calculateCharge(gasUsed, relayRequest.relayData);

balances[relayRequest.relayData.paymaster] = balances[relayRequest.relayData.paymaster].sub(charge);
balances[vars.relayManager] = balances[vars.relayManager].add(charge);
```
위 snippet은 해당 시점까지 소비된 실제 가스가 계산되고 paymaster에 공제되어야 하는 관련 요금이 계산된다. 그리고 이 요금은 paymaster에서 공제되고 중개자와 연결된 해당 중개 관리자에게 추가된다. 이는 중개자가 악의적인 forwarder 주소로 Relay 요청을 생성하고 Vault 소유자가 서명하지 않은 가짜 메시지를 실행하고 여전히 지불을 받을 수 있음을 의미한다.

```
interface GsnTypes {
    /// @notice gasPrice, pctRelayFee and baseRelayFee must be validated inside of the paymaster's preRelayedCall in order not to overpay
    struct RelayData {
        uint256 gasPrice;
        uint256 pctRelayFee;
        uint256 baseRelayFee;
        address relayWorker;
        address paymaster;
        address forwarder;
        bytes paymasterData;
        uint256 clientId;
    }

    //note: must start with the ForwardRequest to be an extension of the generic forwarder
    struct RelayRequest {
        IForwarder.ForwardRequest request;
        RelayData relayData;
    }
}
```
위 코드에서는 두 번째 취약점에 대한 정보가 있다. @notice를 보면 gasPrice, pctRelayFee, baseRelayFee는 paymaster의 `preRelayedCall()` 함수 내부에서 검증되어야 한다고 한다.

```
function preRelayedCall(
    IGsnTypes.RelayRequest calldata _relayRequest,
    bytes calldata,
    bytes calldata,
    uint256
)
external
override
relayHubOnly
returns(bytes memory context_, bool rejectOnRecipientRevert_) {
    address vaultProxy = getParentVault();
    require(
        IVault(vaultProxy).canRelayCalls(_relayRequest.request.from),
        "preRelayedCall: Unauthorized caller"
    );

    bytes4 selector = __parseTxDataFunctionSelector(_relayRequest.request.data);
    require(
        __isAllowedCall(
            vaultProxy,
            _relayRequest.request.to,
            selector,
            _relayRequest.request.data
        ),
        "preRelayedCall: Function call not permitted"
    );

    return (abi.encode(_relayRequest.request.from, selector), false);
}
```
그러나 Enzyme Finance의 paymaster의 `preRelayedCall()` 함수 내부에서는 gasPrice, pctRelayFee, baseRelayFee에 대해서 검증하고 있지 않은 것을 볼 수 있다. 이 문제가 취약한 이유는 RelayHub 내에서 호출되는 `calculateCharge()` 함수를 통해 알 수 있다.

```
function calculateCharge(uint256 gasUsed, GsnTypes.RelayData calldata relayData) public override virtual view returns(uint256) {
    return relayData.baseRelayFee.add((gasUsed.mul(relayData.gasPrice).mul(relayData.pctRelayFee.add(100))).div(100));
}
```
`calculateCharge()` 함수는 위와 같이 구현되어 있다. 요금을 계산할 때, Relay 요청 데이터인 baseRelayFee, gasPrice, pctRelayFee 값을 기반으로 요금을 계산하는 것을 볼 수 있다. 즉, preRelayedCall() 함수에서 gasPrice, pctRelayFee, baseRelayFee에 대한 검증이 없기 때문에 악의적인 중개자는 baseRelayFee 또는 pctRelayFee를 매우 높은 값으로 설정하여 전제 저장소를 고갈 시킬 수 있다.

![](/img/blockchain/asdf2.jpg)

여기서 하나의 제한이 있다. Enzyme Finance의 docs에서 Vault 설정 부분을 보면 gas relayer에는 최대 0.2 eth만 보유할 수 있다는 규약이 있다.

```
uint256 private constant DEPOSIT = 0.5 ether;
(skip)
// PRIVATE FUNCTIONS

/// @dev Helper to pull WETH from the associated vault to top up to the max ETH deposit in the relay hub
function __depositMax() private {
    uint256 prevDeposit = getRelayHubDeposit();

    if (prevDeposit < DEPOSIT) {
        uint256 amount = DEPOSIT.sub(prevDeposit);

        IGasRelayPaymasterDepositor(getParentComptroller()).pullWethForGasRelayer(amount);

        IWETH(getWethToken()).withdraw(amount);

        IGsnRelayHub(getHubAddr()).depositFor {
            value: amount
        }(address(this));

        emit Deposited(amount);
    }
}
```
그러나 위에서 설명한 내용에 따르면 Enzyme Paymaster 컨트랙트 내에 postRelayCall() 함수 내에서는 Relay 요청 데이터의 paymasterData가 true로 설정되어 있다면 `__depositMax()` 함수를 이용해 0.5 eth로 채운다는 것이다. 이를 악용하여 악의적인 중개자는 반복적으로  호출을 하고, 각 호출마다 0.5 ETH를 소모할 수 있다. 금고가 고갈되었음에도 불구하고 이는 다시 채워져 다음 공격에 대비할 수 있다.

---
## 취약점 패치

![](/img/blockchain/asdf3.jpg)

Enzyme Finance 팀은 첫 번째 취약점을 위와 같이 신뢰하는 forwarder 검사를 하는 코드를 추가하여 패치했다.

![](/img/blockchain/asdf4.jpg)

두 번째 취약점은 위와 같이 Relay 요청 데이터인 baseRelayFee와 pctRelayFee의 값을 검증하는 과정을 추가하여 패치했다.

---
## 마무리

이 취약점이 발생한 근본 원인은 결국에 검증해야 할 값을 검증하지 않았기 때문에 발생한 취약점이다. 대부분의 취약점들이 마찬가지지만 이처럼 검사를 하지 않아도 기능상에는 문제가 없지만 보안 이슈로 이어지는 경우가 많이 있다. Smart Contract에서도 취약점이 발생하는 이유는 결국 권한 검증 부재와 같이 검증 부재로 인해서 많은 취약점이 발생한다. 

그리고 Enzyme Finance의 경우에는 GSN라는 오픈소스를 이용하여 사용자가 가스 없이 편리하게 사용하도록 서비스를 구현하는데는 성공하였지만 정해진 지침에 따라 모든 기능을 제대로 구현하지 않았기 때문에 취약점이 발생한 것이다. 이처럼 다른 컨트랙트에서도 오픈소스를 기능적으로만 보고 보안적으로는 제대로 관리하지 않은 것들이 무수히 많을 것이기 때문에 이러한 부분으로 연구를 해가며 취약점을 찾을 수 있을 거 같다.

---
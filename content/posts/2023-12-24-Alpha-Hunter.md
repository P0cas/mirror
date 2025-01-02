---
layout: post
title: '2023 X-mas CTF (Alpha Hunter)'
description: This article is about write-up for the 2023 X-mas CTF. there is only one web3 challenge, which contain length validation bypass and hash collision in discount coupon and underpriced token purchase bug
date: 2023-12-24 02:00:53
author: anonymous
category: 'CTF'
---

## 개요

```
Description
알파벳을 모아 MERRY CHRISTMAS를 완성해 주세요!!

[RPC Usage]
RPC: http://host:port/rpc
GET FLAG: curl -X GET http://host:port/flag

[Account Information]
address: 0x377CFaD82A885Ef59C9243f715F33752804B1126
private key: 0xdba103874bd715cb05989d00d55f53743dbca7c13c77b2b901c4b6ae90232b5e
balance: 1 ETH

[Contract Information]
Setup address: 0x3e8C8ec7F7a5A51a7B4509d2f4d534BB3bA040b1
```
문제를 풀기 위해 제공된 정보는 위와 같다. 컨트랙트는 총 3개가 있는데 Setup 컨트랙트 주소만 제공됐다. (1이더가 제공됨)


```zsh
.
├── House.sol
├── Setup.sol
├── Store.sol
```
제공된 파일은 House, Setup, Store로 총 3개가 제공되었다.

---
## Setup.sol

```
pragma solidity ^0.8.0;

import "./Store.sol";
import "./House.sol";

contract Setup {
    Store public immutable store;
    House public immutable house;
    bool public isStarted;

    constructor() payable {
        store = new Store();
        house = new House();
    }

    function isSolved() external view returns (bool) {
        return 
        store.totalBalances(address(house), "A") >= 1
        && store.totalBalances(address(house), "C") >= 1
        && store.totalBalances(address(house), "E") >= 1
        && store.totalBalances(address(house), "H") >= 1
        && store.totalBalances(address(house), "I") >= 1
        && store.totalBalances(address(house), "M") >= 2
        && store.totalBalances(address(house), "R") >= 3
        && store.totalBalances(address(house), "S") >= 2
        && store.totalBalances(address(house), "T") >= 1
        && store.totalBalances(address(house), "Y") >= 1;
    }
}
```
Setup.sol 파일은 위와 같다. Setup에서 Store, House 컨트랙트를 생성하기 때문에 제공된 Setupt Addr을 통해서 각 컨트랙트의 주소를 가져올 수 있다. 그리고 문제를 해결하기 위한 조건은 house 컨트랙트가 `MERRY CHRISTMAS`라는 문자열을 가지고 있게 하면 풀린다. 해당 문자들은 Store 컨트랙트에서 구매가 가능하고, 이를 타 사용자에게 보낼 수 있다.

---
## House.sol

```
pragma solidity ^0.8.0;

contract House {
}
```
House 컨트랙트는 코드가 없다. 그냥 주소를 위한 컨트랙트이다.

---
## Store.sol

```
pragma solidity ^0.8.0;

contract Store {

    struct DiscountInfo {
        uint256 blockNumber;
        uint[] discountedItems;
        uint[] discountedPrices;
        bytes32 discountCoupon;
    }

    struct Buy {
        uint256[] ids;
        uint256[] prices;
        uint256[] amounts;
    }

    uint256 public constant REGULAR_PRICE = 0.1 ether;
    uint256 public constant PER_N_ITEM = 5;
    bytes public constant  ALPHABETS = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

    // blockNumber -> discountInfo
    mapping (uint256 => DiscountInfo) public discountInfos;
    // coupon -> bool
    mapping (bytes32 => bool) public used; 
    // nonce -> bool
    mapping(uint256 => bool) public nonces;
    // owner => alphabet => amount
    mapping(address => mapping(bytes1 => uint256)) public totalBalances;

    error InvalidRange();
    error AlreadyUsed();
    error NotIssued();
    error NotMatched();
    error LowLevelFailed();

    constructor() {
    }

    // [_lowerBound, _upperBound]
    function randRangeWithNonce(uint256 _blockNumber, uint256 _lowerBound, uint256 _upperBound, uint256 _nonce) internal returns (uint256) {
        if (_lowerBound > _upperBound) revert InvalidRange();
        if (nonces[_nonce]) revert AlreadyUsed();
        nonces[_nonce] = true;
        bytes memory _seeds = (abi.encode(msg.sender, _blockNumber, _lowerBound + _upperBound, _nonce));
        return (uint256(keccak256(_seeds)) % (_upperBound - _lowerBound + 1)) + _lowerBound;
    }

    // allow duplicates
    function randSample(uint256 _blockNumber, uint256[] memory _nonces) internal returns (uint[] memory) {
        uint len = PER_N_ITEM;
        uint[] memory _samples = new uint[](len); 
        for(uint i = 0; i < len; i++) {
            _samples[i] = randRangeWithNonce(_blockNumber, uint8(ALPHABETS[0]), uint8(ALPHABETS[0]) + ALPHABETS.length - 1, _nonces[i]);
        }
        return _samples;
    }

    function getDiscountInfo(uint256 blockNumber) public view returns(uint[] memory, uint[] memory, bytes32) {
        return (discountInfos[blockNumber].discountedItems, discountInfos[blockNumber].discountedPrices, discountInfos[blockNumber].discountCoupon);
    }
    
    function issueDiscountCoupon(uint256 _blockNumber, uint256[] memory _nonce) public returns(uint, uint256[] memory, uint256[] memory, bytes32) {

        DiscountInfo storage _discountInfo = discountInfos[_blockNumber];

        uint256[] memory _prices = new uint256[](PER_N_ITEM);
        uint256[] memory _ids = randSample(_blockNumber, _nonce);

        for(uint256 i = 0; i < PER_N_ITEM; i++) {
            uint256 _price = randRangeWithNonce(_blockNumber, 0.0714285714285 ether, 0.09 ether, _nonce[PER_N_ITEM+i]);
            _prices[i] = _price;
        }

        bytes memory _input = abi.encodePacked(_blockNumber);

        for(uint256 i = 0; i < PER_N_ITEM; i++) {
            _input = abi.encodePacked(_input, _ids[i]);
        }

        for(uint256 id = 0; id < PER_N_ITEM; id++) {
            _input = abi.encodePacked(_input, _prices[id]);
        }

        bytes32 hash = keccak256(_input);

        _discountInfo.blockNumber = _blockNumber;
        _discountInfo.discountedItems = _ids;
        _discountInfo.discountedPrices = _prices;
        _discountInfo.discountCoupon = hash;

        return (_blockNumber, _ids, _prices, hash);
    }

    function buyWithCoupon(bytes32 _coupon, uint256[] memory _ids, uint256[] memory _prices, uint256[] memory _amounts) public payable {
        if (discountInfos[block.number].discountCoupon != _coupon || discountInfos[block.number].discountCoupon == 0) revert NotIssued();
        if (used[_coupon]) revert AlreadyUsed();
        bytes memory _input = abi.encodePacked(block.number);

        for(uint256 i = 0; i < _ids.length; i++) {
            _input = abi.encodePacked(_input, _ids[i]);
        }

        for(uint256 i = 0; i < _prices.length; i++) {
            _input = abi.encodePacked(_input, _prices[i]);
        }

        bytes32 _hash = keccak256(_input);
        if (_hash != _coupon) revert NotMatched();
        used[_coupon] = true;

        uint256 _totalAmount = 0;
        for(uint256 i = 0; i < _ids.length; i++) {
            uint256 _amount = _amounts[i];
            _totalAmount += _prices[i] * _amount;
            totalBalances[msg.sender][bytes1(uint8(_ids[i]))] += _amount;
        }

        require(_totalAmount <= msg.value);

        uint256 left = (msg.value - _totalAmount);
        if (left > 0) {
            (bool success, bytes memory data) = msg.sender.call{value: left}("");
            if(!success) revert LowLevelFailed();
        }

    }

    function buy(uint256[] memory _ids, uint256[] memory _amounts) public payable {

        uint256 _totalAmount = 0;
        for(uint256 i = 0; i < _ids.length; i++) {
            uint256 _amount = _amounts[i];
            _totalAmount += REGULAR_PRICE * _amount;
            totalBalances[msg.sender][bytes1(uint8(_ids[i]))] += _amount;
        }

        require(_totalAmount <= msg.value);

        uint256 left = (msg.value - _totalAmount);
        if (left > 0) {
            (bool success, bytes memory data) = msg.sender.call{value: left}("");
            if(!success) revert LowLevelFailed();
        }
    }

    function resell(uint256 _id, uint256 _amount) public {
        if (totalBalances[msg.sender][bytes1(uint8(_id))] < _amount) _amount = totalBalances[msg.sender][bytes1(uint8(_id))];
        totalBalances[msg.sender][bytes1(uint8(_id))] -= _amount;
        uint256 _amt = _amount * REGULAR_PRICE / 2;
        (bool success, bytes memory data) = msg.sender.call{value: _amt}("");
        if(!success) revert LowLevelFailed();
    }

    function give(uint256 _id, uint256 _amount, address _to) public {       
        if (totalBalances[msg.sender][bytes1(uint8(_id))] < _amount) _amount = totalBalances[msg.sender][bytes1(uint8(_id))];
        totalBalances[msg.sender][bytes1(uint8(_id))] -= _amount;
        totalBalances[_to][bytes1(uint8(_id))] += _amount;
    }

    receive() payable external {}
}
```
Store.sol의 코드는 위와 같다. 함수가 몇 개 없어서 로직 분석은 빠르게 할 수 있다.

```
    struct DiscountInfo {
        uint256 blockNumber;
        uint[] discountedItems;
        uint[] discountedPrices;
        bytes32 discountCoupon;
    }

    struct Buy {
        uint256[] ids;
        uint256[] prices;
        uint256[] amounts;
    }

    uint256 public constant REGULAR_PRICE = 0.1 ether;
    uint256 public constant PER_N_ITEM = 5;
    bytes public constant  ALPHABETS = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

    // blockNumber -> discountInfo
    mapping (uint256 => DiscountInfo) public discountInfos;
    // coupon -> bool
    mapping (bytes32 => bool) public used; 
    // nonce -> bool
    mapping(uint256 => bool) public nonces;
    // owner => alphabet => amount
    mapping(address => mapping(bytes1 => uint256)) public totalBalances;
```
문제에서 사용될 변수들은 위와 같이 정의되어 있다.


```
   function buyWithCoupon(bytes32 _coupon, uint256[] memory _ids, uint256[] memory _prices, uint256[] memory _amounts) public payable {
        if (discountInfos[block.number].discountCoupon != _coupon || discountInfos[block.number].discountCoupon == 0) revert NotIssued();
        if (used[_coupon]) revert AlreadyUsed();
        bytes memory _input = abi.encodePacked(block.number);

        for(uint256 i = 0; i < _ids.length; i++) {
            _input = abi.encodePacked(_input, _ids[i]);
        }

        for(uint256 i = 0; i < _prices.length; i++) {
            _input = abi.encodePacked(_input, _prices[i]);
        }

        bytes32 _hash = keccak256(_input);
        if (_hash != _coupon) revert NotMatched();
        used[_coupon] = true;

        uint256 _totalAmount = 0;
        for(uint256 i = 0; i < _ids.length; i++) {
            uint256 _amount = _amounts[i];
            _totalAmount += _prices[i] * _amount;
            totalBalances[msg.sender][bytes1(uint8(_ids[i]))] += _amount;
        }

        require(_totalAmount <= msg.value);

        uint256 left = (msg.value - _totalAmount);
        if (left > 0) {
            (bool success, bytes memory data) = msg.sender.call{value: left}("");
            if(!success) revert LowLevelFailed();
        }

    }
```
`buyWithCoupon()` 함수는 생성된 쿠폰을 이용해서 문자를 구매하는 함수이다. `issueDiscountCoupon()` 함수에서 생성된 쿠폰을 기반으로 원하는 수량만큼 구매한다.

```
    function buy(uint256[] memory _ids, uint256[] memory _amounts) public payable {

        uint256 _totalAmount = 0;
        for(uint256 i = 0; i < _ids.length; i++) {
            uint256 _amount = _amounts[i];
            _totalAmount += REGULAR_PRICE * _amount;
            totalBalances[msg.sender][bytes1(uint8(_ids[i]))] += _amount;
        }

        require(_totalAmount <= msg.value);

        uint256 left = (msg.value - _totalAmount);
        if (left > 0) {
            (bool success, bytes memory data) = msg.sender.call{value: left}("");
            if(!success) revert LowLevelFailed();
        }
    }
```
`buy()` 함수는 위와 같다. 우리가 원하는 알파벳을 살 수 있다. 코드를 보면 알파벳 하나당 0.1 ether에 구매할 수 있는 것을 확인할 수 있다. 그러나 우리가 구매해야하는 총 문자 개수는 MERRY CHRISTMAS로 14개이다. 현재 우리에게 제공된 잔액은 1 ether인데 `buy()` 함수를 이용해서 모든 알파벳을 구매하려면 1.4 ehter + gas가 필요하기 때문에 구매를 할 수 없다.

```
    function give(uint256 _id, uint256 _amount, address _to) public {       
        if (totalBalances[msg.sender][bytes1(uint8(_id))] < _amount) _amount = totalBalances[msg.sender][bytes1(uint8(_id))];
        totalBalances[msg.sender][bytes1(uint8(_id))] -= _amount;
        totalBalances[_to][bytes1(uint8(_id))] += _amount;
    }
```
`give()` 함수는 위와 같다. 우리가 구매한 알바벳을 타 주소로 전달할 수 있다. 즉, 우리는 MERRY CHRISTMAS에 포함되어 있는 모든 문자를 구매 후에 `give()` 함수를 통해서 house 컨트랙트 주소로 넘기면 된다.

```
function issueDiscountCoupon(uint256 _blockNumber, uint256[] memory _nonce) public returns(uint, uint256[] memory, uint256[] memory, bytes32) {

        DiscountInfo storage _discountInfo = discountInfos[_blockNumber];

        uint256[] memory _prices = new uint256[](PER_N_ITEM);
        uint256[] memory _ids = randSample(_blockNumber, _nonce);

        for(uint256 i = 0; i < PER_N_ITEM; i++) {
            uint256 _price = randRangeWithNonce(_blockNumber, 0.0714285714285 ether, 0.09 ether, _nonce[PER_N_ITEM+i]);
            _prices[i] = _price;
        }

        bytes memory _input = abi.encodePacked(_blockNumber);

        for(uint256 i = 0; i < PER_N_ITEM; i++) {
            _input = abi.encodePacked(_input, _ids[i]);
        }

        for(uint256 id = 0; id < PER_N_ITEM; id++) {
            _input = abi.encodePacked(_input, _prices[id]);
        }

        bytes32 hash = keccak256(_input);

        _discountInfo.blockNumber = _blockNumber;
        _discountInfo.discountedItems = _ids;
        _discountInfo.discountedPrices = _prices;
        _discountInfo.discountCoupon = hash;

        return (_blockNumber, _ids, _prices, hash);
    }
```
`issueDiscountCoupon()` 함수는 전달받은 블록 번호와 nonce 값을 기반으로 쿠폰을 생성한다. 이 쿠폰이 생성될 때 할일을 받을 알파벳과 할인율은 nonce 값과 블록 번호에 의해서 생성된다. 쿠폰 번호는 nonce와 블록 번호를 이용해서 구한 _ids + _prices를 모두 바이트 변환하여 더한 값을 해시로 생성한다.

```
    function randSample(uint256 _blockNumber, uint256[] memory _nonces) internal returns (uint[] memory) {
        uint len = PER_N_ITEM;
        uint[] memory _samples = new uint[](len); 
        for(uint i = 0; i < len; i++) {
            _samples[i] = randRangeWithNonce(_blockNumber, uint8(ALPHABETS[0]), uint8(ALPHABETS[0]) + ALPHABETS.length - 1, _nonces[i]);
        }
        return _samples;
    }
```
할인을 받을 알파벳은 `randSample()` 함수로 생성된다. `randSample()` 함수 내부를 보면 `ransRangeWithNonce()` 함수로 ids 값을 만드는데 보면 nonce 값을 기반으로 알바벳을 구하고 있다.

```
randRangeWithNonce(_blockNumber, 0.0714285714285 ether, 0.09 ether, _nonce[PER_N_ITEM+i]);
```
할인율을 구하는 로직은 `randRangeWithNonce()` 함수를 위와 같이 호출한다. 위 함수 호출을 통해서 생성될 수 있는 할인 금액의 범위는 0.07 ~ 0.09 ehter이다. 만약 운 좋게 모든 알바벳의 할인 금액이 최저 금액으로 설정 된다고 하더라도 0714285714285 * 14로 0.9999999999989999 ether이다.

구매를 해야하는 총 알바벳은 14개이고, `buyWithCoupon()` 함수에 의해서 구매되는 알바벳은 5개이다. (쿠폰 생성 시에 5개를 생성하기 때문에) 즉, 요청을 3번 나누어서 5, 5, (4 + 1)로 1번, 2번 호출에서는 필요한 알파벳 10개를 구매하도록 하고, 3번째 호출에서 4개는 우리가 필요한 알파벳 마지막 1개는 아무 알파벳을 사도록 한다면 총 15개의 쿠폰을 사는 것이기에 잔액 부족으로 트랜잭션이 거부될 것이다.

```
   function buyWithCoupon(bytes32 _coupon, uint256[] memory _ids, uint256[] memory _prices, uint256[] memory _amounts) public payable {
        if (discountInfos[block.number].discountCoupon != _coupon || discountInfos[block.number].discountCoupon == 0) revert NotIssued();
        if (used[_coupon]) revert AlreadyUsed();
        bytes memory _input = abi.encodePacked(block.number);

        for(uint256 i = 0; i < _ids.length; i++) {
            _input = abi.encodePacked(_input, _ids[i]);
        }

        for(uint256 i = 0; i < _prices.length; i++) {
            _input = abi.encodePacked(_input, _prices[i]);
        }

        bytes32 _hash = keccak256(_input);
        if (_hash != _coupon) revert NotMatched();
        used[_coupon] = true;

        uint256 _totalAmount = 0;
        for(uint256 i = 0; i < _ids.length; i++) {
            uint256 _amount = _amounts[i];
            _totalAmount += _prices[i] * _amount;
            totalBalances[msg.sender][bytes1(uint8(_ids[i]))] += _amount;
        }

        require(_totalAmount <= msg.value);

        uint256 left = (msg.value - _totalAmount);
        if (left > 0) {
            (bool success, bytes memory data) = msg.sender.call{value: left}("");
            if(!success) revert LowLevelFailed();
        }

    }
```
`buyWithCoupon()` 함수를 다시 보자. 컨트랙트 내에 저장되어 있는 쿠폰 번호와 현재 블록 번호와 전달받은 _ids, _prices 값을 기반으로 생성된 쿠폰 값을 비교하여 일치하는지 확인하고 있다. 그러나 이 함수 내에서 _ids, _prices 값에 대해서 길이 검증이 존재하지 않는다. 쿠폰 생성 시에는 _ids, _prices가 각 각 5개로 고정이다. 

```
_ids = [1, 1, 1, 1, 1]
_prices = [1, 1, 1, 1, 1]

_ids = [1, 1]
_prices = [1, 1, 1, 1, 1, 1, 1, 1]
```
먼저 알아야 할 내용은 첫 번째 값과 두 번째 값의 해시 값은 동일하다는 것이다. 이유는 쿠폰을 생성할 때, 각 값을 단순하게 이전 바이트 값에 현재 바이트 값을 더해서 해시화하기 때문이다. 이를 이용하면 구매할 때 꼭 5개의 알파벳을 사지 않고, 우리가 원하는 개수만큼 알파벳을 구매할 수 있다. _ids에서 사지 않을 알파벳은 _prices로 옮기면 된다.

> 공격 시나리오

1. `randRangeWithNonce()` 함수를 통해서 우리가 구해야 하는 알파벳들과 일치하는 Nonce 값을 찾는다.
2. Step-1에서 찾은 Nonce 값을 2개 또는 3개로 쿠폰을 생성한다.
3. 생성한 쿠폰으로 알바펫을 사고(2개 또는 3~4개), 이를 `give()` 함수로 house로 보낸다.

---
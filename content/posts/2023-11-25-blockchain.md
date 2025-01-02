---
layout: post
title: 'Basic knowledge of web3.0'
description: This article is about the blockchain
date: 2023-11-25 02:00:53
author: anonymous
category: Theory
math: true
pin: true
---
# 개요

오늘부터 바쁜 일들이 조금씩 끝나서 1년 동안 미루고 있던 web3.0 공부를 시작하려고 한다. 처음 보는 분야인 만큼 시간을 투자해서 기초를 튼튼히 다질 계획이다.

---
## 블록체인이란

![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F9a5130d6-cf56-4bd9-8e9b-4ac666fd88f6%2F0b9180cd-967f-4b6d-826c-82c5e3cf54bf%2FUntitled.png?table=block&id=03f24911-2b55-43d0-88f9-60ec677f99e2&spaceId=9a5130d6-cf56-4bd9-8e9b-4ac666fd88f6&width=2000&userId=54bb8d41-f1b7-45fb-8f3a-7162d9846226&cache=v)

블록체인은 쉽게 말해서 변조 방지를 위한 디지털 장부라고 볼 수 있다.  블록체인은 위와 같은 블록의 체인이며 블록 내에는 거래 내역 및 데이터들이 저장되고, 이 블록 내에 저장되는 데이터는 블록체인의 유형에 따라 다르다. 또한 이 블록체인은 중앙화로 구성되어 있는 은행과는 다르게 탈중앙화 되어 있다.

![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F9a5130d6-cf56-4bd9-8e9b-4ac666fd88f6%2F55476596-d523-4c54-9f9c-585048c908a2%2FUntitled.png?table=block&id=8ae0ca9f-6b6f-40c8-a81b-ffdbe97b9335&spaceId=9a5130d6-cf56-4bd9-8e9b-4ac666fd88f6&width=2000&userId=54bb8d41-f1b7-45fb-8f3a-7162d9846226&cache=v2)

중앙화의 경우에는 하나의 중앙 서버에 모든 데이터를 저장하고 있기 때문에 중앙 서버가 해킹 당하면 사용자의 데이터가 변조 되고 삭제될 수 있다. 그러나 탈중앙화의 경우 네트워크의 접속되어 있는 모든 노드가 데이터를 공유하고 있기 때문에 하나의 노드가 해킹을 당하더라도 해커는 정보를 변조하거나 삭제할 수 없다. 이것이 탈중앙화의 주 목적이다.

![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F9a5130d6-cf56-4bd9-8e9b-4ac666fd88f6%2F3ad2a4e7-8343-46e3-a03a-43c11d5eb77b%2FUntitled.png?table=block&id=fef43e6c-7c8d-4f2e-a703-8406c2c64975&spaceId=9a5130d6-cf56-4bd9-8e9b-4ac666fd88f6&width=2000&userId=54bb8d41-f1b7-45fb-8f3a-7162d9846226&cache=v2)

또 다른 예로는 구글의 Spreadsheet을 예로 들 수 있다. Spreadsheet을 누군가 생성을 하고, 이를 공유하면 네트워크를 통해 공유된다. 이렇게 공유된 네트워크에 사람들이 링크를 가지고 있는 누구나 접속을 해서 Spreadsheet을 읽고, 수정할 수 있다. 만약 누군가 Spreadsheet의 내용을 변경한다면 이는 이 네트워크에 접속하고 있는 모든 사용자의 컴퓨터에서 업데이트가 된다. 그러나 이때 누군가 한 명이라도 업데이트를 원치 않는다면 업데이트 되지 않는다.  블록체인도 마찬가지다. 하나의 블록이 해킹을 당해 장부의 데이터가 변조 되었다고 해도 장부를 가지고 있는 모든 사람이 동의해야 하기 때문에 변조되지 않는다.

---
## 블록체인의 중요 기능

1. **탈중앙화**
2. **분산 원장**
3. **불변성**
4. **합의**
5. **이중 지출의 문제 해결**

블록체인에 있어 중요한 기능을 하는 요소는 탈중앙화와 분산 원장, 불변성, 합의, 이중 지출의 문제 해결이다.

---
## 블록체인의 구조

![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F9a5130d6-cf56-4bd9-8e9b-4ac666fd88f6%2F70117f31-da8b-4bd0-bed8-4e8e92350a64%2FUntitled.png?table=block&id=33fc5068-5c1a-4da6-9793-188f601617a4&spaceId=9a5130d6-cf56-4bd9-8e9b-4ac666fd88f6&width=2000&userId=54bb8d41-f1b7-45fb-8f3a-7162d9846226&cache=v2)

일단 블록체인에서의 첫 번째로 생성되는 블록은 Genesis Block이라고 한다. Genesis Block에는 장부의 데이터가 존재하지 않는다.

![](https://www.researchgate.net/publication/337306138/figure/fig1/AS:825909783310336@1573923645768/The-structure-of-a-Blockchain-A-block-is-composed-of-a-header-and-a-body-where-a-header.png)

그리고 두 번째 블록부터는 Index, Hash, Hash of the Previous Block, Mercle Root, Time-Stamp, Nonce로 구성되어 있는 Block-Header와 Data & Transactions로 구성되어 있는 Block-Body로 구성된다.

> Block Header

`Index`는 현재 블록체인에서의 블록의 순서를 나타내는 번호이다.

`Hash`는 블록의 고유 Hash이다. 블록체인의 모든 블록은 각 고유의 Hash를 가진다. 만약 블록을 변조하면 Hash 또한 변하게 되고 이 경우에는 해당 블록은 블록체인에 포함되지 않는다. 그러나 이 경우라도 블록은 삭제되지 않는다. 블록체인에 한 번 추가된 블록은 변경하거나 삭제할 수 없다.

`Hash of the Previous Block`은 이전 블록의 Hash이다. 모든 블록은 이전 블록의 해시를 가지고 있다. 이 때문에 블록이 변경되어 해시가 변경된 경우에 이 해시가 변경되었다고 판단할 수 있다.

`Mercle Root`는 Block-Body에 있는 Transaction의 머클 트리 루트의 해시 값으로 블록 내의 모든 Transaction을 요악한 데이터이다.

`Timestamp`는 블록이 생성된 시간이라 보면 된다.

`Nonce`는  블록 번호,  데이터, 이전 Hash와 함께 블록의 유효한 Hash를 계산하기 위한 해싱 알고리즘에 사용될 정수이다.

> Block Body

`Data & Transactions`은 거래와 관련된 정보가 저장된다. 예를 들면 보낸/받는 사람 정보, 금액과 같은 정보들이다.

```javascript
const SHA256 = require('crypto-js/sha256');
class CryptoBlock{
    constructor(index, timestamp, data, precedingHash=" "){
        this.index = index;
        this.timestamp = timestamp;
        this.data = data;
        this.previousHash = previousgHash;
        this.hash = this.computeHash();
    }
    computeHash(){
        return SHA256(this.index + this.precedingHash + this.timestamp + JSON.stringify(this.data)).toString();
    }
}
```
블록 생성을 위한 클래스는 위와 같이 만들 수 있다 (머클 루트는 제외 함). index는 현재 블록의 번호, timestampe는 블록이 생성되는 시간, data는 거래와 관련된 정보, previousHash에는 이전 블록의 해시, hash는 현재 블록의 해시 값이다. 그리고 현재 블록의 해시를 생성하는 메서드인 computeHash() 메서드를 보면 현재 블록의 번호, 이전 블록의 해시, 현재 블록의 생성 시간, 거래 정보를 모두 더해서 해시로 만드는 것을 확인할 수 있다.

---
## 블록체인의 작동 방식

![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F9a5130d6-cf56-4bd9-8e9b-4ac666fd88f6%2F8032fad4-dbac-4ae5-ae33-3ab957a333c2%2FUntitled.png?table=block&id=dabaabba-2b8a-4953-b24a-ab26222e96b4&spaceId=9a5130d6-cf56-4bd9-8e9b-4ac666fd88f6&width=2000&userId=54bb8d41-f1b7-45fb-8f3a-7162d9846226&cache=v2)

블록체인의 작동 방식은 먼저 사용자가 거래를 요청하면 블록이 생성되는데 이 과정에서 PoW 작업을 거쳐 특정 조건을 만족하는 유효한 블록의 해시 값을 생성하게 되고, 이때 생성된 해시가 블록의 고유 해시이다. 이후에는 브로드캐스트에 의해서 전달받은 블록은 네트워크의 모든 노드로 전달되고, 모든 노드에서  검증을 맞친 이후에 아무런 문제가 없다면 이를 블록체인에 연결한다.

```javascript
const crypto = require("crypto");
const { range } = require("lodash");
const SHA256 = (buffer) => {
    return crypto.createHash('sha256').update(buffer).digest('hex')
}

class CryptoBlock {
    constructor(index, timestamp, data, precedingHash = " ") {
        this.index = index;
        this.timestamp = timestamp;
        this.data = data;
        this.precedingHash = precedingHash;
        this.hash = this.computeHash();
        this.nonce = 0;
    }

    computeHash() {
        return SHA256(
            this.index +
            this.precedingHash +
            this.timestamp +
            JSON.stringify(this.data) +
            this.nonce
        ).toString();
    }

}

class CryptoBlockchain {
    constructor() {
        this.blockchain = [this.startGenesisBlock()];
        this.difficulty = 4;
    }
    startGenesisBlock() {
        return new CryptoBlock(0, "01/01/2020", "Init", "0");
    }

    obtainLatestBlock() {
        return this.blockchain[this.blockchain.length - 1];
    }

    addNewBlock(newBlock) {
        newBlock.precedingHash = this.obtainLatestBlock().hash;
        this.blockchain.push(newBlock);
    }

    checkChainValidity() {
        for (let i = 1; i < this.blockchain.length; i++) {
            const currentBlock = this.blockchain[i];
            const precedingBlock = this.blockchain[i - 1];

            if (currentBlock.hash !== currentBlock.computeHash()) {
                return false;
            }
            if (currentBlock.precedingHash !== precedingBlock.hash) return false;
        }
        return true;
    }
}

let smashingCoin = new CryptoBlockchain();

console.log("smashingCoin mining in progress....");

for(let i = 1; i < 4; i++){
    smashingCoin.addNewBlock(
        new CryptoBlock(i, "01/06/2020", {
            sender: "Pocas",
            recipient: "Tom",
            quantity: 50
        })
    );
}

console.log(JSON.stringify(smashingCoin, null, 4));
```
위 코드는 위에서 설명한 블록체인에 블록이 추가되기 까지의 과정을 구현한 코드이다.

![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F9a5130d6-cf56-4bd9-8e9b-4ac666fd88f6%2F44a1e496-8b63-4230-bc9a-e700f4476645%2F%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-11-28_22.18.06.png?table=block&id=06986ef7-77dc-4b39-b5ed-dc0b896dea57&spaceId=9a5130d6-cf56-4bd9-8e9b-4ac666fd88f6&width=2000&userId=54bb8d41-f1b7-45fb-8f3a-7162d9846226&cache=v2)

그러나 코드를 실행해보면 두 번째, 세 번째, 네 번째 블록의 해시를 보면 합의 알고리즘이 적용되지 않은 것을 볼 수 있다. 이 경우, 블록체인 네트워크는 노드들 간에 일관된 데이터 상태를 유지하는 데 어려움을 겪을 수 있다. 합의 알고리즘은 분산된 환경에서 동일한 블록체인을 유지하기 위해 필요한 규칙과 메커니즘을 제공한다. 따라서 합의 알고리즘이 없다면 일관성 부재, 데이터 충돌과 위조, 이중 지불과 같이 보안 이슈가 발생할 수 있다.

```javascript
proofOfWork(difficulty) {
        while (
            this.hash.substring(0, difficulty) !== Array(difficulty + 1).join("0")
        ) {
            this.nonce++;
            this.hash = this.computeHash();
        }
    }
```
위와 같은 함수를 추가하고 다시 코드를 실행 해보자.

![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F9a5130d6-cf56-4bd9-8e9b-4ac666fd88f6%2Fe254e2ae-8076-45b1-bfa1-169db6949cb9%2FUntitled.png?table=block&id=3e90ab38-73d7-4eaa-b415-881d2a677d97&spaceId=9a5130d6-cf56-4bd9-8e9b-4ac666fd88f6&width=2000&userId=54bb8d41-f1b7-45fb-8f3a-7162d9846226&cache=v2)

블록의 해시를 보면 0이 4개로 시작하는 것을 볼 수 있다. 즉 nonce 값을 이용해서 해시를 만드는데 만약  difficulty(난이도)와 충족되는 해시가 아니라면 nonce의 값을 증가 시켜서 충족되는 해시가 나올 때 까지 루프를 타게 된다.

위 코드는 따로 난이도 조절은  없고, 예시로 든 코드이지만 실제로 블록체인 네트워크는 일정한 간격으로 새로운 블록을 생성하는 것을 목표로 하고 있기 때문에 난이도 조절을 한다.

---
## 합의 알고리즘

합의 알고리즘은 단일 노드나 노드 그룹이  네트워크를 조작할 수 없도록 하여 블록체인의 무결성을 유지하는 역할을 한다. 쉽게 말해 이는 블록체인 네트워크에 합의를 달성하는 기술이라고 볼 수 있다.

---
## 작업 증명 (Proof of Work, PoW)

블록체인의 합의 알고리즘 중 하나인 작업 증명은 특정한 작업을 통해 무언가 증명한다라는 의미를 가지고 있다. 이 과정에서 해시가 생성되는데 해시를 생성하기 전에 특정한 작업을 통해 조건에 맞는 논스 값을 찾는다. 이때 논스 값을 찾을 때 난이도라는 것이 있는데 난이도에 따라서 논스 값을 찾는 시간이 오래 걸릴 수도 있고, 적게 걸릴 수도 있다.

```plaintext
A : Block 0000f67ff4f28c53dc1a53d0cc5a18cc469d0266293df522f772c95b9a053a08
B : Block 00000003cc073bd06073fa9a10aa2fc6dbcb1451ce11e57aa059957612c1ab71
```
A 블록의 앞 부분을 보면 0이 4개 B 블록의 앞 부분을 보면 0이 6개가 있는 것을 볼 수 있다. 이것이 난이도이다. 난이도가 늘어날 수록 해시 앞 부분이 0 * 난이도의 수와 일치하는 해시의 논스 값을 찾아야 한다.

```plaintext
Bitcoin Block
818,405 : 00000000000000000003dfee45ec31f096acca2a933df216bd373b800dac11dd

Bitcoin Block
818,403 : 00000000000000000004be497352098e30b362433f37d07140bcaa8454020b58
0000000000000000000379e25d3695e0ae78e6e7c95c6f1afd676ed20fc7ac94
```
위의 해시는 최신 비트코인 블록의 해시 값이다.

```plaintext
CryptoBlock {
  index: 1,
  timestamp: 1700909310,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: '6b8467d6436d83fd1e22e9660d90ef8b35792dacfa75a1d08406d3ebdc08cbc4',
  hash: '721c2426199b3527ea4b000a4d0e27374b6dc11cdae4c2415b2a37721c8f55a9',
  difficulty: 0,
  nonce: 0
}
CryptoBlock {
  index: 2,
  timestamp: 1700909310,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: '721c2426199b3527ea4b000a4d0e27374b6dc11cdae4c2415b2a37721c8f55a9',
  hash: '0d2733eb1bfe02921da7abc60e5f1f756985a5934aac43bd762aed3bcb17296f',
  difficulty: 0,
  nonce: 0
}
CryptoBlock {
  index: 3,
  timestamp: 1700909310,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: '0d2733eb1bfe02921da7abc60e5f1f756985a5934aac43bd762aed3bcb17296f',
  hash: 'fd169f49e3313a6a3ed0daed78ea4b242af8025182a609f8689717c5f1be7646',
  difficulty: 0,
  nonce: 0
}
CryptoBlock {
  index: 4,
  timestamp: 1700909310,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: 'fd169f49e3313a6a3ed0daed78ea4b242af8025182a609f8689717c5f1be7646',
  hash: '689c8bcefaad4320ff7ffee387de834479f4605ffef3ce739f2ce7afbb7958d0',
  difficulty: 0,
  nonce: 0
}
CryptoBlock {
  index: 5,
  timestamp: 1700909310,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: '689c8bcefaad4320ff7ffee387de834479f4605ffef3ce739f2ce7afbb7958d0',
  hash: 'dcc7ef1c10eff0d20da8cdc5642de7b4b914aba8690d0eb22ecbc58535dd91e3',
  difficulty: 0,
  nonce: 0
}
CryptoBlock {
  index: 6,
  timestamp: 1700909310,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: 'dcc7ef1c10eff0d20da8cdc5642de7b4b914aba8690d0eb22ecbc58535dd91e3',
  hash: 'd5ad47ba4d90c23cb055aeeb785d27cf4910e1dc6008e9230987d514d9a52786',
  difficulty: 0,
  nonce: 0
}
CryptoBlock {
  index: 7,
  timestamp: 1700909310,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: 'd5ad47ba4d90c23cb055aeeb785d27cf4910e1dc6008e9230987d514d9a52786',
  hash: '733a5bef6f8430bf55efaab6a25cabd2d1e3b13a083f630f9c7f062325bbfc4e',
  difficulty: 0,
  nonce: 0
}
CryptoBlock {
  index: 8,
  timestamp: 1700909310,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: '733a5bef6f8430bf55efaab6a25cabd2d1e3b13a083f630f9c7f062325bbfc4e',
  hash: '0885eed98f9582156424ec2e4c826c0c0119d1d5e5ce035cb221cbc53025bcff',
  difficulty: 0,
  nonce: 0
}
CryptoBlock {
  index: 9,
  timestamp: 1700909310,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: '0885eed98f9582156424ec2e4c826c0c0119d1d5e5ce035cb221cbc53025bcff',
  hash: '3681b23f553ec6dd56e65ee4131250b0d589e4a648975a6034f514c321e6fbd8',
  difficulty: 0,
  nonce: 0
}
CryptoBlock {
  index: 10,
  timestamp: 1700909310,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: '3681b23f553ec6dd56e65ee4131250b0d589e4a648975a6034f514c321e6fbd8',
  hash: '1bfacdf37afd8e72a8b8edb6941bd3b0974cb1291e2c388a71415ca9b55ec4ca',
  difficulty: 0,
  nonce: 0
}
1
CryptoBlock {
  index: 11,
  timestamp: 1700909310,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: '1bfacdf37afd8e72a8b8edb6941bd3b0974cb1291e2c388a71415ca9b55ec4ca',
  hash: '04da5d9eba6af3ef0285fe3564a901bf1e44aab332a2e95ede22f92b2d997042',
  difficulty: 1,
  nonce: 2
}
CryptoBlock {
  index: 12,
  timestamp: 1700909310,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: '04da5d9eba6af3ef0285fe3564a901bf1e44aab332a2e95ede22f92b2d997042',
  hash: '0924f09d6f6ffaeba0e14a71094916576a07883bc734c4ac95e0f54ad27d62e2',
  difficulty: 1,
  nonce: 3
}
```
위는 예상 채굴 시간과 난이도 조절 수를 각 10으로 설정한 후 PoS를 해서 블록을 생성한 결과이다. 블록의 난이도를 보면 10번째까지는 난이도가 0이였다가 10 이후부터는 난이도가 올라간 것을 볼 수 있다.

```
CryptoBlock {
  index: 69,
  timestamp: 1700909400,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: '00000054e898e4cddd4d4d0e836b4cc270d3016d3194531ccb1f1f82c1202610',
  hash: '0000000f7449f9c543756aa037d21bc3d828d3986da022217fbded98276785ea',
  difficulty: 6,
  nonce: 7487662
}
CryptoBlock {
  index: 70,
  timestamp: 1700909409,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: '0000000f7449f9c543756aa037d21bc3d828d3986da022217fbded98276785ea',
  hash: '00000005456f6faceed5d54926425ea07aa2e6cfe7556b8de115f129817568db',
  difficulty: 6,
  nonce: 34368702
}
CryptoBlock {
  index: 71,
  timestamp: 1700909449,
  data: { sender: 'Pocas', recipient: 'Tom', quantity: 50 },
  precedingHash: '00000005456f6faceed5d54926425ea07aa2e6cfe7556b8de115f129817568db',
  hash: '00000003cc073bd06073fa9a10aa2fc6dbcb1451ce11e57aa059957612c1ab71',
  difficulty: 7,
  nonce: 88725539
}
```
이후 69, 70, 71 블록을 보면 난이도가 생성하여 논스 값을 최대 88725539번을 돌려서 찾은 것을 확인할 수 있다. 이처럼 난이도가 올라가면 조건의 맞는 논스 값을 찾는데 오래 걸린다.

---
## 지분 증명 (Proof of Stake, PoS)

PoS은 PoW(Proof of Work)보다 전력 소비량이 적고 블록 생성 속도가 빠르며, 보안성을 유지하는 장점이 있다. PoS를 간단히 설명하면 자신이 갖고 있는 암호화폐의 양에 따라서 블록을 생성할 권한을 준다.

PoS는 PoW(Proof of Work)와 달리 블록을 채굴하는 대신, 네트워크 참여자들이 자신들의 암호화폐를 스테이킹하여 네트워크 보안을 유지하고 새로운 블록을 생성한다. PoS에서는 블록을 생성할 때 랜덤성을 사용하는 것이 일반적이다. 이때 랜덤성은 보유한 코인의 양과 관련이 있는데 일반적으로 블록을 생성할 때 스테이킹된 코인의 양이 클수록 블록을 생성할 확률이 높아지게 된다.

---
## 머클루트

머클루트는 블록의 헤더에 포함되어 있으며, 블록바디에 있는 모든 transaction의 요약본이라고 보면 된다. 조금 더 정확히 말하면 transactions의 머클트리이다.

![](https://www.forex.academy/wp-content/uploads/2020/05/Merkle-Tree-FA.jpg)

모든 블록의 해시를 확인하는 것은 매우 비효율적이고 시간 소모적이다. 머클트리는 데이터 무결성을 확인하는데 있어 효율적이기 때문에 사용된다. 먼저 머클 루트가 생성되는 방법은 위 사진을 기반으로 T<sub>A &nbsp;</sub>는 A의 transaction이고, T<sub>B &nbsp;</sub>는 B의 transaction이다. 그리고 각 트랜잭션의 해시는 H<sub>A &nbsp;</sub>, H<sub>B &nbsp;</sub>로 나타낸다. 이때 제일 가까운 두 개의 해시가 합쳐지는데 합쳐진 해시는 H<sub>AB &nbsp;</sub>가 된다. 이런 식으로 하나 하나 합쳐 나가면서 최종적으로 하나의 해시가 완성되는데 그 해시가 머클 루트이다. 위 사진에서는 H<sub>ABCDEFGH &nbsp;</sub>인데 이는 T<sub>A &nbsp;</sub>부터 T<sub>H &nbsp;</sub>까지의 거래의 머클 루트이다. 그리고 이렇게 생성된 머클 루트에 의해서 불변성과 무결성을 쉽게 유지할 수 있다.


```
Merkle Root of 99,997 : 5140e5972f672bf8e81bc189894c55a410723b095716eaeec845490aed785f0e

TXID 0 : b86f5ef1da8ddbdb29ec269b535810ee61289eeac7bf2b2523b494551f03897c
TXID 1 : 80c6f121c3e9fe0a59177e49874d8c703cbadee0700a782e4002e87d862373c6
```
99,997번 비트코인의 머클루트는 Merkle Root of 99,997이고, 이 블록에는 총 2개의 거래가 있다. 각 TXID는 TXID 0,1이다. 이 TXID를 이용해 위 머클루트를 구해보자.

$$ MerkleRoot=Hash(Hash(Concatenate(A 
1
​
 ,A 
2
​
 ))+Hash(Concatenate(A 
3
​
 ,A 
4
​
 ))) $$

머클트리를 만드는 공식은 위와 같다.

```javascript
const crypto = require('crypto');

function hexToBin(hexString) {
    return Buffer.from(hexString, 'hex');
}

function sha256(input) {
    const hash = crypto.createHash('sha256').update(Buffer.from(input, 'hex')).digest();
    return crypto.createHash('sha256').update(hash).digest().toString('hex')
}

function littleEndianToBigEndian(input) {
    const buf = Buffer.from(input, 'hex');
    const reversedBuf = Buffer.from(buf.reverse());
    return reversedBuf.toString('hex');
}

function bigEndianToLittleEndian(input) {
    const buf = Buffer.from(input, 'hex');
    const reversedBuf = Buffer.from(buf.reverse());
    return reversedBuf.toString('hex');
}

big_hash_0 = littleEndianToBigEndian("b86f5ef1da8ddbdb29ec269b535810ee61289eeac7bf2b2523b494551f03897c")
big_hash_1 = littleEndianToBigEndian("80c6f121c3e9fe0a59177e49874d8c703cbadee0700a782e4002e87d862373c6")

console.log(big_hash_0)
console.log(big_hash_1)

console.log(bigEndianToLittleEndian(sha256(big_hash_0 + big_hash_1)))
// 5140e5972f672bf8e81bc189894c55a410723b095716eaeec845490aed785f0e
```
리틀 엔디안을 빅 엔디안으로 변환해주는 것이 핵심이다. 빅 엔디안으로 변환해준 이유는 비트코인에서는 거래 데이터의 대부분의 필드가 리틀 엔디안 형식이기 때문이다. 리틀 엔디안에서 빅 엔디안으로 변환할 때 먼저 각 문자 쌍을 바꾼 다음 문자열을 반대로 바꿔준다.

```python
import hashlib
import binascii

u=lambda x: binascii.unhexlify(x)
r=lambda x: x[::-1]

r(u("b86f5ef1da8ddbdb29ec269b535810ee61289eeac7bf2b2523b494551f03897c"))
```
파이썬에서는 위와 같이 리틀 엔디안에서 빅 엔디안으로 변환할 수 있다.

```
❯ node Merkle\ root.js
Merkle Root : 5140e5972f672bf8e81bc189894c55a410723b095716eaeec845490aed785f0e
```
코드를 돌려보면 머클루트가 정상적으로 구해지는 것을 확인할 수 있다.

---
# 이더리움이란

이더리움은 스마트 계약이라 불리는 애플리케이션 코드를 안전하게 실행하고, 확인하는 P2P 네트워크를 구축하는 분산형 블록체인 플랫폼이, 화폐단위는 ETH이다. 스마트 계약을 통해서는 사용자가 중앙 기관 없이 서로 거래를 할 수 있다. 이더리움의 특징은 Smart Contract, EVM, DAPPs, DAOs이다.

---
## 이더리움 계정

![](https://media.geeksforgeeks.org/wp-content/uploads/20220721160225/1235311.jpg)

이더리움 네트워크에는 계정이라는 것이 존재한다. 계정의 기본 구성 요소는 Nonce, Ether Balance, Contract Code, Storage, Code Hash가 있다.

계정의 유형은 외부 소유 계정인 EOA, 컨트랙트 계정인 CA로 2개의 계정이 있으며, EOA -> EOA, EOA -> CA로는 transaction을 전송할 순 있지만 CA -> EOA로는 transaction을 자체적으로 전송할 수 없다. 컨트랙트 계정에서 외부 소유 계정으로의 트랜잭션 전송을 제한하는 이유는 보안과 컨트롤 관련 문제 때문이다.

> 외부 소유 계정 (EOA)

- 주소(Address): 20바이트 크기의 해시로, 이더리움에서 자산을 보관하거나 송금을 받을 수 있는 식별이다. 주소는 일반적으로 '0x'로 시작하는 40자리 16진수 형태로 표현된다.
- 비밀키(Private Key): 계정을 제어하고 서명하여 트랜잭션을 인증하는 데 사용되는 개인 키다. 보안에 매우 중요한 정보이며, 외부 소유 계정을 제어하기 위해 필요하다.

EOA는 개인이나 사용자가 소유하는 계정으로, 이더리움의 주요 지갑(Wallet)을 대표하며, EOA는 주소와 비밀키로 구성되어 있다.

> 컨트랙트 계정 (CA)

- 주소(Address): 스마트 계약의 주소는 컨트랙트가 배포된 위치를 식별한다. 이 주소로 트랜잭션을 보내거나 다른 계정과 상호작용할 수 있다
- 컨트랙트 코드(Contract Code): 스마트 계약은 배포된 코드를 포함하고 있으며, 이 코드는 계약의 동작 및 상태를 정의한다

이더리움 블록체인에 배포된 스마트 컨트랙트를 나타내며 계약 주소와 컨트랙트 코드로 구성되어 있다.

---
## 이더리움 가상 머신 (EVM)
![](https://eattheblocks.com/wp-content/uploads/2021/11/evm.png)

EVM은 Ethereum Virtual Machine의 줄임말로 스마트 컨트랙트 코드 및 Dapp을 실행하기 위한 런타임 환경이다. EVM의 주요 구조는 스택, 스토리지, 스마트 컨트랙트 코드, 가스 및 비용 실행, 스마트 컨트랙트 실행 환경이 있다. `스택`에는 함수 호출 및 매개변수, 임시 변수, 메모리 구조/참조와 같은 연산에 필요한 데이터가 저장되며 컨트랙트가 종료되면 스택은 모두 사라진다. `스토리지`에는 계약 상태 변수, 계약 내부 데이터와 같은 영구적인 상태를 저장하는데 사용된다. `가스 및 실행 비용`은 Gad Limit, Gas Price와 같은 트랜잭션 수수료에 관련된 값이 관리된다.

이 EVM 내에서는 가스라는 일종의 수수료를 지불하게 되는데 이 가스가 사용되는 이유는 블록체인 네트워크의 안정성, 무결성, 그리고 자원의 효율적 사용을 보장하기 위해서다. 대표적인 예로는 DDoS 공격을 방지할 수 있다. 만약 공격자가 여러 트랜잭션을 생성하여 네트워크를 공격을 할 경우에는 과도한 가스 비용을 지불해야 하므로 DDoS(분산 서비스 거부) 공격을 방지할 수 있게 된다. 이외에도 네트워크 과부화 방지, 리소스 효율성 등 다양한 이유가 있다.

---
## 스마트 컨트랙트

![](https://assets-global.website-files.com/63120caf05c416571b0a4bac/633544114ee1009d16a8aad7_620cf5f9674588a56f7a4c88_how-smart-contract-works-ulam-labs.png)

Smart Contract는 서면으로 작성되는 계약을 코드화 하여 특정 조건이 충족될 때 거래가 진행되는 스크립트이며 [Solidity](https://soliditylang.org/), [Vyper](https://docs.vyperlang.org/en/stable/), Move와 같은 Smart Contract 전용 프로그래밍 언어로 작성된다. 일반적으로 은행에서 송금을 하게 되면 은행을 거쳐서 다른 사람에게 송금을 하게 된다. 그러나 스마트 컨트랙트는 제3자(은행과 같은) 없이 합법적이고 투명한 방식으로 금전, 재산, 주식 또는 가치 있는 모든 것을 교환하고 그에 따른 높은 수수료를 지불하는 기본을 확립한다.

1. 컴파일: 스마트 컨트랙트는 Solidity와 같은 언어로 작성되며, 이를 컴파일러를 통해 바이너리 코드로 변환해야 한다. 컴파일은 소스 코드를 블록체인이 이해할 수 있는 형태로 변환하는 과정이다.
2. 배포(Deployment): 컴파일된 스마트 컨트랙트는 블록체인 상에 배포된다. 이때 특정 주소를 가지며, 블록체인 네트워크의 모든 노드에 복제한다.
3. 트랜잭션 발생: 스마트 컨트랙트를 실행하기 위해 트랜잭션이 발생한다. 이 트랜잭션은 스마트 컨트랙트의 함수를 호출하거나, 데이터를 전송하는 등의 작업을 수행한다.
4. 트랜잭션 처리: 네트워크의 노드들은 발생한 트랜잭션을 검증하고 블록에 포함시킵니다. 이때 실행되는 스마트 컨트랙트의 함수는 조건을 확인하고, 필요한 작업을 수행한다.
5. 상태 변경: 스마트 컨트랙트의 실행으로 인해 블록체인의 상태가 변경된다 이는 데이터의 변경, 잔액의 이동 등의 작업을 의미한다.
6. 결과 반환: 스마트 컨트랙트의 실행이 완료되면 결과가 반환된다. 이 결과에는 트랜잭션의 성공 여부, 실행된 함수의 결과 등이 포함될 수 있다.

스마트 컨트랙트의 실행 방식은 위와 같다. 그러나 스마트 컨트랙트의 조건이 충족되지 않으면 트랜잭션이 실패하게 되고, 변경 사항이 블록체인에 기록되지 않는다. 예를 들어, A가 B에게 이더리움을 전송하면, 스마트 컨트랙트는 이 조건을 확인하고 거래를 실행한다. 이때, 조건이 충족되지 않으면 거래는 실패하게 된다.

그리고 이 기술은 현재 Finance, Insurance, Digital Identity, Other Uses: Supply Chain와 같은 다양한 곳에서 사용되고 있다.

---
## 스마트 컨트랙트의 보안성

Smart Contract는 사용자가 원하는 계약의 조건을 Solidity와 같은 언어를 이용해 코드로 작성해서 블록체인 네트워크에 배포한다. 결국 이 Smart Contract는 사용자가 작성한 코드이기 때문에 취약점이 있을 수 밖에 없다. 또한 Smart Contract는 블록체인의 특성인 불변성에 따라서 한 번 배포된 이후에는 변경될 수 없기 때문에 Smart Contract를 작성할 때는 신중해야 한다.

이 Smart Contract에서 발생할 수 있는 취약점은 Reentrancy, Missing Access Controls, Weak Access Controls, Privilege Escalation, Governance를 포함한 다양한 취약점이 있다.

### Reentrancy Attacks

재진입 공격은 일반적으로 스마트 컨트랙트 내부에서 외부 컨트랙트나 함수를 호출할 때 발생하는데, 트랜잭션이 끝나기 전에 공격자가 함수 호출을 재진입하여 컨트랙트의 상태를 변경할 수 있는 취약점이다. (자금 유출, 상태 변조, Dos와 같은 것들로 이어질 수 있다.)

```
pragma solidity ^0.8.0;

contract Vulnerable {
    mapping(address => uint) public balances;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint _amount) public {
        require(balances[msg.sender] >= _amount, "Insufficient balance");
        (bool success, ) = msg.sender.call{value: _amount}("");
        require(success, "Transfer failed");
        balances[msg.sender] -= _amount;
    }
}
```
위의 코드는 간단한 계정 입출금을 처리하는 스마트 컨트랙트이다. `deposit()` 함수를 호출하여 잔액을 입금하고, `withdraw()` 함수를 호출하여 잔액을 출금할 수 있다. 위 코드를 분석해 보면 하나의 취약점이 있다. `withdraw()` 함수에서 잘못된 방식으로 외부로 돈을 출금하고 있다. 구현되어 있는 로직을 보면 돈을 먼저 출금한 이후에 사용자의 잔액을 차감하고 있다. 이 경우에 트랜잭션이 끝나기 전에 공격자는 `withdraw()` 함수를 재호출할 수 있다.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2Fc889fbc8-ca9c-47cd-91fb-07fcd1ab987c_1518x801.png)

재진입 공격을 구성도로 나타내면 위와 같다. 재진입 공격에 취약한 스마트 컨트랙트는 잔액 출금 트랜잭션이 끝난 이후에 잔액을 차감한다. 그러나 출금 트랜잭션이 끝나기 전에 `fallback()` 또는 `receive()` 함수를 통해서 취약한 컨트랙트의 `withdraw()` 함수를 재호출함으로 잔액이 차감되기 전에 다시 컨트랙트 내에 있는 자금을 해커의 잔액만큼 계속 가져올 수 있다. (해커의 잔액이 0으로 업데이트되지 않았기 때문에 출금이 되었더라도 현재 잔액은 그대로 저장되어 있음.)

```
contract Attacker {
    Vulnerable vulnerable;

    constructor(address _vulnerableAddress) {
        vulnerable = Vulnerable(_vulnerableAddress);
    }

    function attack() public payable {
        vulnerable.deposit{value: msg.value}();
        vulnerable.withdraw(msg.value);
    }

    fallback() external payable {
        if (address(vulnerable).balance >= msg.value) {
            vulnerable.withdraw(msg.value);
        }
    }
}
```
재진입 공격 코드는 위와 같이 작성할 수 있다. 취약한 컨트랙트의 `deposit()` 함수를 이용해서 이더를 송금한 이후에 withdraw() 함수를 이용해서 잔액을 출금한다. 이때 취약한 컨트랙트에서 Attacker 컨트랙트로 돈을 송금하면 `fallback()` 함수가 호출된다. 이 `fallback()` 함수 내에서 바로 취약한 컨트랙트의 withdraw() 함수를 재호출해서 잔액을 다시 출금할 수 있다. 번외로 [The Dao](https://www.gemini.com/cryptopedia/the-dao-hack-makerdao)라는 보안계의 한 획을 그은 사건이 있다. 해커가 재진입 공격을 이용해 6천만 달러의 이더리움을 탈취한 사건이다.

---
## ERC 토큰이란

이더리움에서 토큰은 스마트 컨트랙트를 기반으로 발행된 디지털 자산을 의미한다. 쉽게 말하면 암호화폐와 같은 맥락으로 보면 될 거 같다. 이더리움 블록체인 위에서 독자적인 토큰이나 가상 자산을 만들어내기 위해서는 `ERC-20`, `ERC-721`, `ERC-1155`와 같은 토큰 표준을 준수하는 스마트 컨트랙트를 개발해야한다. 이 토큰들은 이더리움에서 거의 모든 것을 나타낼 수 있는데 게임 내에서 캐릭터의 스킬이나 복권 또는 온라인 플랫폼에서 평판 점수와 같이 모든 것에 대해서 토큰을 발행해 사용할 수 있다.


### ERC-20

ERC-20은 대체 가능한 토큰에 대한 표준이다. 이 ERC-20은 각 토큰을 다른 토큰으로 동일하게 만들 수 있는 속성이 있다.

- 사용자로부터 다른 사용자에게 토큰을 전송
- 사용자의 현재 잔액 확인
- 현재 네트워크에 사용 가능한 총 공급량 확인
- 제 3자에게 토큰을 양도할 수 있도록 승인

ERC-20의 기능은 위와 같이 제공한다.

```
function name() public view returns (string)
function symbol() public view returns (string)
function decimals() public view returns (uint8)
function totalSupply() public view returns (uint256)
function balanceOf(address _owner) public view returns (uint256 balance)
function transfer(address _to, uint256 _value) public returns (bool success)
function transferFrom(address _from, address _to, uint256 _value) public returns (bool success)
function approve(address _spender, uint256 _value) public returns (bool success)
function allowance(address _owner, address _spender) public view returns (uint256 remaining)

event Transfer(address indexed _from, address indexed _to, uint256 _value)
event Approval(address indexed _owner, address indexed _spender, uint256 _value)
```
[EIP-20](https://eips.ethereum.org/EIPS/eip-20)에서 제공되는 메서드/이벤트는 위와 같으며, 만약 Smart Contract가 위와 같은 메서드로 구현이 되어 있다면 이건 ERC-20 컨트랙트라고 보면 된다.

### ERC-721

ERC-721은 우리가 흔히 알고 있는 NFT의 표준이다. NFT는 Non-Fungible Token으로 대체 불가능한 토큰이라는 의미이다. 이 NFT는 대체가 불가능한만큼 수집품, 액세스 키, 복권, 콘서트 및 스포츠 경기의 번호가 매겨진 좌석 등을 제공하는 플랫폼에서 사용하기 적합하다. 또한 NFT에는 TokenId라는 uint256 변수가 있으므로 ERC-721 계약의 경우 쌍 계약 주소인 uint256 tokenId는 전역적으로 고유해야 한다.

```
function balanceOf(address _owner) external view returns (uint256);
function ownerOf(uint256 _tokenId) external view returns (address);
function safeTransferFrom(address _from, address _to, uint256 _tokenId, bytes data) external payable;
function safeTransferFrom(address _from, address _to, uint256 _tokenId) external payable;
function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
function approve(address _approved, uint256 _tokenId) external payable;
function setApprovalForAll(address _operator, bool _approved) external;
function getApproved(uint256 _tokenId) external view returns (address);
function isApprovedForAll(address _owner, address _operator) external view returns (bool);

event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);
event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId);
event ApprovalForAll(address indexed _owner, address indexed _operator, bool _approved);
```
위 메서드/이벤트들은 [EIP-712](https://eips.ethereum.org/EIPS/eip-721)에서 제공되는 메서드 및 이벤트들이다. 제공되는 메서드를 보면 EIP-20과 동일한 메서드도 있지만 대부분은 다른 것을 확인할 수 있다.

> 번외로 ERC-712의 _safeMint() 함수는 토큰을 생성 하기 전에 수신하려는 주소가 ERC-712 토큰을 수신할 수 있는지 확인하는 과정을 거친다. OpenZeppelin의 공식 문서에 따르면 수신 주소가 Smart Contract이면 this.onERC721Received.selector를 반환하는 onERC721Received()라는 함수가 구현되어 있어야 한다고 한다.
{: .prompt-info }

```
    function _safeMint(address to, uint256 tokenId, bytes memory data) internal virtual {
        _mint(to, tokenId);
        _checkOnERC721Received(address(0), to, tokenId, data);
    }
```
ERC721.sol 파일을 확인해 보면 `_safeMint()` 함수 내에서 `_checkOnERC721Received()` 함수를 호출해서 수신자가 토큰 전송을 수락하는지 또는 주소가 컨트랙트인지 확인한다.

```
    function _checkOnERC721Received(address from, address to, uint256 tokenId, bytes memory data) private {
        if (to.code.length > 0) {
            try IERC721Receiver(to).onERC721Received(_msgSender(), from, tokenId, data) returns (bytes4 retval) {
                if (retval != IERC721Receiver.onERC721Received.selector) {
                    revert ERC721InvalidReceiver(to);
                }
            } catch (bytes memory reason) {
                if (reason.length == 0) {
                    revert ERC721InvalidReceiver(to);
                } else {
                    /// @solidity memory-safe-assembly
                    assembly {
                        revert(add(32, reason), mload(reason))
                    }
                }
            }
```
`_checkOnERC721Received()` 함수는 위와 같이 구현되어 있다.


### ERC-1155

ERC-1155는 여러 유형의 토큰 타입을 관리하는 표준이다. 배포된 단일 컨트랙트에는 대체 가능한/불가능한 토큰 또는 다른 설정들의 조합이 포함될 수 있다. 이 ERC-1155는 ERC-20과 ERC-712 토큰의 기능을 동일하게 할 수 있으며 동시에 수행할 수도 있다.

- 하나의 호출에서 여러 잔액 전송
- 하나의 호출에서 여러 자산의 잔액 확인
- 모든 사용자에게 토큰 승인
- 토큰 hook 수신
- 총 공급량이 하나라면 이는 NFT로 대함
- 안전한 전송 룰 설정

ERC-1155에서 제공하는 함수 및 기능은 위와 같다.

```
// ERC-20
function transferFrom(address from, address to, uint256 value) external returns (bool);

// ERC-1155
function safeBatchTransferFrom(
    address _from,
    address _to,
    uint256[] calldata _ids,
    uint256[] calldata _values,
    bytes calldata _data
) external;
```
`safeBatchTransferFrom()` 함수는 한 번 호출을 통해서 여러 잔액을 한 번에 전송할 수 있도록 하는 함수이다. ERC-20의 `transferFrom()` 함수를 보면 from, to, value로 3개의 인자를 받지만, ERC-1155의 `safeBatchTransferFrom()` 함수를 보면 _ids와 _values를 배열로 전송해서 데이터를 전송하는 것을 볼 수 있다.

```
// ERC-20
function balanceOf(address owner) external view returns (uint256);

// ERC-1155
function balanceOfBatch(
    address[] calldata _owners,
    uint256[] calldata _ids
) external view returns (uint256[] memory);

// ERC-1155
function setApprovalForAll(
    address _operator,
    bool _approved
) external;

function isApprovedForAll(
    address _owner,
    address _operator
) external view returns (bool);

```
`balanceOfBatch()`, `isApprovedForAll()`, `setApprovalForAll()` 함수도 이하 동일

```
function onERC1155BatchReceived(
    address _operator,
    address _from,
    uint256[] calldata _ids,
    uint256[] calldata _values,
    bytes calldata _data
) external returns(bytes4);
```
ERC-165 지원을 고려할 때, ERC-1155에서는 Smart Contract에 대해서 수신 후크를 지원한다.

- 호출자는 _from 주소에 대해서 토큰 사용이 승인되어 있어야 하고 그렇지 않은 경우 호출자와 _from은 같아야 한다.
- 다음의 경우 트랜잭션을 되돌린다.
    1. _to 주소가 0인 경우
    2. _ids의 길이와 _values의 길이가 같이 않은 경우
    3. _ids의 토큰 보유자의 잔액이 보내려는 잔액 보다 작은 경우
    4. 아무 에러가 발생 했을 경우

ERC-1155에서는 안전한 transfer에 대해서 위와 같은 규칙이 있다.

---
## Solidity

### Private Variable

```
Everything you use in a smart contract is publicly visible, even local variables and state variables marked private.
```
Solidity에서 Private 변수를 선언한다고 해도 이 변수가 외부에서 읽을 수 없다는 것은 아니다.


### Devision

```
Since the type of the result of an operation is always the type of one of the 
operands, division on integers always results in an integer. In Solidity, division 
rounds towards zero. This mean that int256(-5) / int256(2) == int256(-2).
```
Solidity 공식 문서에서 나누기에 대한 설명은 위와 같이 다루고 있다. 정수를 나눌 경우 반환 값은 항상 정수로 된다는 건데, 이는 나눗셈의 결과가 0.009와 같은 소수가 나온다면 이는 0으로 반올림 된다는 것을 알 수 있다.


### tx.origin

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1660372147633/c9w4VjaCM.png?auto=compress,format&format=webp)

`tx.origin`은 이더리움 Smart Contract 내에서 사용되는 전역 변수 중 하나이다. 이 변수는 트랜잭션을 발생시킨 계정의 주소가 저장된다. 이 변수는 주로 `msg.sender`와 비교하여 사용이 되는데 `msg.sender`는 현재 함수를 호출한 계정 또는 스마트 계약의 주소를 나타내고, `tx.origin`은 최초 트랜잭션을 발생 시킨 주체의 주소를 나타낸다. 그래서 위 사진을 보면 `tx.origin`의 값은 최초 트랜잭션을 발생 시킨 계정의 주소로 고정되어 있는 것을 확인할 수 있다.

### DelegateCall

![](https://blockchain-academy.hs-mittweida.de/wp-content/uploads/2021/09/delegatecall_context-1024x422.jpg)

`DelegateCall()` 함수는 `call()` 함수와 거의 비슷한데 다른 점은 컨텍스트에 있다. `DelegateCall()` 함수를 사용한다면 컨텍스트가 보존되어 msg.sender, msg.value가 변하지 않는다. 

![](https://substackcdn.com/image/fetch/w_2400,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F06b55334-ee0d-43b6-bb4f-a871e4f83d3e_1584x571.png)

그리고 target contract에서 실행되어 값이 변한다고 해도 이 값은 caller contract 스토리지에 저장된다.

---
## Foundry

### forge script
forge-script - 스마트 계약을 스크립트로 실행하여 온체인으로 보낼 수 있는 트랜잭션을 구축할 수 있다.

```zsh
~/Desktop/web3.0/web3.0-foundry/test master*
❯ tree          
.
└── Solve.sol

1 directory, 1 file

~/Desktop/web3.0/web3.0-foundry/src master*
❯ tree
.
├── exploit.sol
└── interfaces

2 directories, 1 file
```
나는 위와 같이 구성해서 사용한다. Solve.sol에서 exploit.sol 내에 있는 공격용 컨트랙트를 불러와서 사용한다.

```
forge script --broadcast --rpc-url $RPC_URL --private-key $PRIVATE_KEY Solve -v
```
명령어는 간단하게 위와 같이 사용한다. 미리 환경 변수로 rpc url과 private key를 등록한 이후에 사용

```
--broadcast
    transactions을 브로드캐스트

--skip-simulation
    on-chain simulation을 건너뜀

--target-contract contract_name
    내가 실행하고 싶은 컨트랙트를 선택할 수 있음

--with-gas-price price
    가스 설정 가능

--private-key raw_private_key
    제공된 개인 키를 사용

-v 
    EVM의 상세 정보 출력 (v 개수마다 상세 정보가 다름)
    - 2: 모든 테스트에 대한 로그를 출력
    - 3: 실패한 테스트에 대한 실행 추적을 출력
    - 4: 모든 테스트에 대한 실행 추적 및 실패한 테스트에 대한 설정 추적을 출력
    - 5: 모든 테스트에 대한 실행 및 설정 추적을 출력
```

### cast

작성 예정

---
## ethers.js

- new ethers.Wallet(pvkey, rpc_url) // signer 생성
- new ethers.Contract(contract_address, abi, signer) // 컨트랙트 생성
- signer.sendTransaction({to:owner, value : 100000000000000}) // 이더 송금
- provider.getBlockNumber() // 현재 블록 번호
- ethers.utils.formatEther(string) // 출력을 wei 대신 ether로
- ethers.utils.parseEther(string) // ether를 wei로 변환
- getStorage(contract_address, slot) // slot 데이터 가져오기
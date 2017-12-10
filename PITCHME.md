# Ethereum Made Simple

## E.1 목차

* E.2 Ethereum
* E.3 네트워크
* E.4 계정
* E.5 거래
* E.6 Smart Contract 개발-컴파일-거래-배포-사용
* E.7 Smart Contract 사용 geth, curl, ethjsonrpc (python), nodejs, web
* E.8 이벤트

---
## 문제 e-1: 처음 Geth 네트워크 구성하고 접속하기

### genesis.json 설정

* '최초' geth 네트워크를 구성하려면, genesis 설정으로 초기화 한다.
    * 자신만 사용하는 local private network을 구성한다.
    * live network은 ether로 실제 거래할 수 있는 네트워크를 말한다.

* genesis.json을 설정한다.
    * private으로 설정하는 경우, Frontier guide의 genesis.json 예제를 사용한다.
        https://github.com/ethereum/go-ethereum/wiki/Private-network
---
키 | 설명
-------|-------
nounce | 4바이트, proof-of-work의 암호 hash를 위해 무작위로 생성된 임의의 수. 난이도보다 보통 낮다
alloc | 충전 금액을 적s고 네트워크를 만들면 된다 (내 account address) (지금은 되지 않는 듯하다) <br>"alloc": {"0xaddress": { "balance": "amount denoted in Wei" }}

---     


```python
%%writefile _CustomGenesis.json
{
    "nonce": "0x0000000000000042",
    "timestamp": "0x0",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "extraData": "0x0",
    "gasLimit": "0x8000000",
    "difficulty": "0x400",
    "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "coinbase": "0x3333333333333333333333333333333333333333",
    "alloc": {}
}
```

    Overwriting _CustomGenesis.json
---

### bash shell을 사용해서 geth 네트워크를 실행한다.

* init명령어를 사용해서 방금 작성한 genesis.json 설정으로 실행한다.


```python
%%writefile _geth_CustomGensis.sh
_dir=$HOME/Downloads/eth/1
geth --datadir $_dir init _CustomGenesis.json
```

    Overwriting _geth_CustomGensis.sh
---

!sh _geth_CustomGensis.sh

* 새로운 계정을 만들면 충전을 해야 거래를 발생할 수 있다.
    * 충전을 하려면 계정을 새로 생성하고, 마이닝을 하면 된다 (아니면 직접 구매를 해야 한다).
    * personal.newAccount("password")
    * miner.start(1);admin.sleepBlocks(1);miner.stop()



```python
!geth --datadir ~/Downloads/eth/1 account new
```

## 문제 e-2: Geth 네트워크 구성하기 (ip를 넣어서)

* ip를 사용하여 private network구성하고, 접속하기

* Geth 서버


```python
%%writefile _geth3.sh
_dir=$HOME/Downloads/eth/2
_log=$HOME/Downloads/eth/2.log
geth --identity "myNode" \
--rpc --rpcaddr "117.xxx.xxx.xxx" --rpcport "8001" --rpccorsdomain "*" \
--datadir $_dir \
--port "30301" \
--nodiscover \
--ipcapi "admin,db,eth,debug,miner,net,shh,txpool,personal,web3" \
--rpcapi "db,eth,net,web3" \
--networkid 33 \
console 2>>$_log
```

* shell에서 실행 (노트북에서는 console이 실행되지 않는다)
    ```
    $ sh _geth3.sh
    ```


선택 | 설명 | 기본 값
-----------|-----------|-----------
--networkid | 네트워크 인식번호, 임의의 양수 | 0=Olympic, 1=Frontier, 2=Morden
--identity "jslNode" | 자신의 private chain 명칭 |
--nodiscover | 다른 노드가 자동으로 찾지 못하게 함. |
--maxpeers | 다른 노드의 최대 접속 수. 0으로 하면 다른 노드는 사용 허용하지 않음. |
--unlock | 계정의 잔고를 사용할 수 있게 잠금 해제 | 
--rpc | http-rpc를 사용할 수 있게 함 | enabled
--rpcapi | http-rpc에서 사용할 수 있는 api (예: db,eth,net,web3) | web3
--rpcaddr | is the interface you want to listen on for conenctions. 0.0.0.0 is all interfaces |
--rpcport | is the port it will listen on | 8080 (geth)
--rpccorsdomain | rpc 요청을 할 수 있는 도메인 (URL) "*"는 어떤 도메인이나 허용 |
--datadir | private chain 데이터를 저장하는 디렉토리 |
--port "30303" | "network listening port" |
--verbosity | how much of the inner working of Geth are shown | 0=silent, 1=error, 2=warn, 3=info, 4=core, 5=debug, 6=debug detail
--mine | 마이닝 |

## 문제 e-3: Geth 멀티노드 네트워크 구성하기 (ip를 넣어서)

* e-2를 멀티노드로 변경한다.
* 멀티노드multinode는 복수의 노드를 생성해서, peer로 추가해야 한다.
    * private node를 만들고 (117.xxx.xxx.xxx)
    * 원격에서 특정 ip(또는 도메인만 사용허용)

* 주의
    * network id가 동일해야 한다
    * genesis가 동일해야 한다.


* cluster를 구성한다.
    * 첫번째 노드는 아래 shell을 실행한다.
        ```
        sh _geth1.sh
        ```
    
    * 두번째 노드는 아래 shell을 실행한다.
        ```
        sh _geth2.sh
        ```
        * 그리고 enode값을 구한다.
            ```
            > admin.nodeInfo.enode
            "enode://e0...@[::]:30006"
            ```
    
    * 두번째 노드를 첫번째에 추가한다. ([::]를 ip주소로 바꾸고)
    ```
    admin.addPeer("enode://...@117.xxx.xxx.xxx:30006?discport=0")
    ```
    
    * configure permannent static nodes
        * bootnodes를 정해 놓으면, static-nodes.json 없어도 됨.
        ```
        <datadir>/static-nodes.json
        ```


* 첫번째 노드를 실행한다.


```python
# %load _geth1.sh
_dir=$HOME/Downloads/eth/1
_log=$HOME/Downloads/eth/1.log
geth --identity "myNode" \
--unlock 0 \
--verbosity 3 \
--rpc --rpcaddr "117.xxx.xxx.xxx" --rpcport "8xxx" --rpccorsdomain "*" \
--datadir $_dir \
--port "30005" \
--bootnodes "enode://e0...@117.xxx.xxx.xxx:30006" \
--maxpeers 3 \
--rpcapi "admin,db,eth,debug,miner,net,shh,txpool,personal,web3" \
--ipcdisable \
--networkid 11 \
console 2>>$_log

```


```python
# %load $HOME/Downloads/eth/1/static-nodes.json
[
    "enode://e0...@117.xxx.xxx.xxx:30006",
    "enode://00...@117.xxx.xxx.xxx:30005"
]

```

* 두번째 노드를 실행한다.


```python
# %load _geth2.sh
_dir=$HOME/Downloads/eth/1
_log=$HOME/Downloads/eth/1.log
geth --identity "jslNode" \
--unlock 0 \
--verbosity 3 \
--rpc --rpcaddr "117.xxx.xxx.xxx" --rpcport "8xxx" --rpccorsdomain "*" \
--datadir $_dir \
--port "30006" \
--bootnodes "enode://00...@117.xxx.xxx.xxx:30005" \
--maxpeers 3 \
--rpcapi "admin,db,eth,debug,miner,net,shh,txpool,personal,web3" \
--ipcdisable \
--networkid 11 \
console 2>>$_log

```

* 멀티노드를 성공적으로 구성하면, peer접속 정보를 볼 수 있다.

!geth --exec "admin.peers;" attach http://117.xxx.xxx.xxx:8xxx

## E.6 Smart Contract 개발

### E.6.1 Smart Contract이란?

* 분산네트워크에서 실행되는 분산데이터와 프로그램
* 비트코인의 블럭체인을 확장. 비트코인은 데이터만을 저장하지만, 이더리움은 프로그램도 저장.
* 분산프로그램을 Smart Contract이라고 한다.
* 사용할 때는 운영체제 evm을 설치해서 smart contract을 사용.

### E.6.2 절차

* Ethereum Virtual Machine에서 contract을 개발한다.

구분 | 설명 | 결과
-----|-----|-----
개발 | 언어로 contract 구현 | 소스
컴파일 | 소스를 evm 바이너리코드가 생성 | abi, bin (code)
거래 | 거래를 생성하고 마이닝 대기 | transactionHash
배포 (마이닝) | 마이닝하고 bloack chain의 주소 획득 | contractAddress
사용 | abi, contractAddress로 contract api 호출 | contract api 결과

* 1-1 개발
    * 언어를 선택하여 (solidity), 프로그램을 구현한다.
    * 온라인 ide [browser-solidity](https://ethereum.github.io/browser-solidity/)
        * 'create'
            * (인자가 있으면 넣고) 'create'를 누른다.
            * 누르고 나면, contract생성
        * '함수'
            * setter함수에 필요한 args를 타입에 맞게 넣고, '함수 버튼'을 누른다.
            * gettter함수로 출력

* 1-2 컴파일
    * solc로 컴파일 할 수 있다.
    * solc 설치되어 있지 않으면 browser-solidity를 사용한다.

* 1-3 거래
    * 거래가 발생하면, transactionHash가 생성된다.
    * 마이닝 전까지 대기 pending

* 1-4 배포 (마이닝과 같은 의미)
    * contract 배포 요청, 마이닝, blockchain에 주소를 받기 까지의 과정을 말한다.
    * blockchain에 배포하고 나면, 수정불가능 (immutable)
    * 배포하려면 mining을 해야 한다.
    * 마이닝 시작
        * geth console
        ```
        > miner.start(1);admin.sleepBlocks(1);miner.stop
        ```

    * 마이닝 완료 
        * 블록체인에 배포된다.
        * blockNumber +1 증가.
        * 주소 값이 생성된다. 마이닝 완료 전 null이다.
            * geth console
            ```
            > eth.getTransactionReceipt("0x3c58a...50bd5")
            ```         

* 1-5 사용
    * contract api 서비스를 호출한다.
        ```
        var greeter = eth.contract(ABI).at(Address);
        ```


## 문제 e-6: solidity를 컴파일하기 (solc)

* 개발-컴파일-거래-배포-사용 사이클
    * solidity 개발-컴파일

### 컴파일러 준비

* 컴파일러를 Solidity로 설정한다.
* 설정된 Solidity로 컴파일 한다.

* 사용할 노드에 컴파일러 설정되었는지 확인
    ```
    > eth.getCompilers()
    ```

### greeter.sol

* 출처 https://www.ethereum.org/greeter
* 복수 contract을 컴파일하고, 서로 참조하는 경우


```python
%%writefile src/greeter.sol
pragma solidity ^0.4.6;
contract mortal {
    /* Define variable owner of the type address*/
    address owner;

    /* this function is executed at initialization and sets the owner of the contract */
    function mortal() { owner = msg.sender; }

    /* Function to recover the funds on the contract */
    function kill() { if (msg.sender == owner) selfdestruct(owner); }
}

contract greeter is mortal {
    /* define variable greeting of the type string */
    string greeting;

    /* this runs when the contract is executed */
    function greeter(string _greeting) public {
        greeting = _greeting;
    }

    /* main function */
    function greet() constant returns (string) {
        return greeting;
    }
}
```

    Overwriting src/greeter.sol



```python
res=!solc --optimize --bin src/greeter.sol
mycompiled=res.fields()[3][0]
print mycompiled
```

    60606040523461000057604051610284380380610284833981016040528051015b5b60008054600160a060020a0319166c01000000000000000000000000338102041790555b8060019080519060200190828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061009157805160ff19168380011785556100be565b828001600101855582156100be579182015b828111156100be5782518255916020019190600101906100a3565b5b506100df9291505b808211156100db57600081556001016100c7565b5090565b50505b505b610192806100f26000396000f3606060405260e060020a600035046341c0e1b58114610029578063cfae321714610038575b610000565b34610000576100366100b3565b005b34610000576100456100f5565b60405180806020018281038252838181518152602001915080519060200190808383829060006004602084601f0104600302600f01f150905090810190601f1680156100a55780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b6000543373ffffffffffffffffffffffffffffffffffffffff908116911614156100f25760005473ffffffffffffffffffffffffffffffffffffffff16ff5b5b565b604080516020808201835260008252600180548451600282841615610100026000190190921691909104601f8101849004840282018401909552848152929390918301828280156101875780601f1061015c57610100808354040283529160200191610187565b820191906000526020600020905b81548152906001019060200180831161016a57829003601f168201915b505050505090505b9056


## E.7 Smart Contract 사용

* geth console
* json rpc (ethjsonrpc)
* nodejs
* web pages


## 문제 e-7: geth console

* multiply
* greeter


### 단순한 session


```python
%%writefile src/e_test2.js
var primary = web3.eth.accounts[0];
var bal=web3.eth.getBalance(primary)
var c=web3.net.peerCount;
console.log(bal);
console.log(c);
console.log(eth.blockNumber);
console.log(txpool.status.pending);
console.log('net.listening: ',net.listening);
console.log('net.peerCount: ', net.peerCount);
```

    Overwriting src/e_test2.js



```python
!geth --exec 'loadScript("src/e_test2.js")' attach http://117.xxx.xxx.xxx:8xxx
```

### multiply

* geth console에서의 사용 예.
```
> myMultiply=eth.contract(compiled.test.info.abiDefinition).at(_object.address)
{
  abi: [{
      constant: false,
      inputs: [{...}],
      name: "multiply",
      outputs: [{...}],
      payable: false,
      type: "function"
  }],
  address: "0x050c96a751f86a699a541001223b1ba7f5364d8d",
  transactionHash: null,
  allEvents: function(),
  multiply: function()
}
> myMultiply.multiply.call(6)
'42'
```


```python
%%writefile src/e_testGeth.js
var primary = eth.accounts[0];
console.log("primary ac: ",primary);
console.log("balance: ",eth.getBalance(primary));
source = "contract test {\n" + "   /// @notice will multiply `a` by 7.\n" + "   function multiply(uint a) returns(uint d) {\n" + "      return a * 7;\n" + "   }\n" + "} ";
    "contract test {\n   /// @notice will multiply `a` by 7.\n   function multiply(uint a) returns(uint d) {\n      return a * 7;\n   }\n} ";
compiled = eth.compile.solidity(source);
_class=eth.contract(compiled.test.info.abiDefinition);
console.log('bin code: ', compiled.test.code)
//async way
_object=_class.new({from:primary,data:compiled.test.code}, function(err, contract) {
  if (!err && contract.address)
    console.log("contractAddress: ", contract.address);
    console.log("transactionHash: ", contract.transactionHash);
    myMultiply=_class.at(contract.address);
    console.log("multiply: ",myMultiply.multiply.call(6));
});
```

    Overwriting src/e_testGeth.js



```python
!geth --exec 'loadScript("src/e_testGeth.js")' attach http://117.xxx.xxx.xxx:8xxx
```


```python
!geth --exec 'eth.pendingTransactions;' attach http://117.xxx.xxx.xxx:8xxx
```

    [{
        blockHash: [1mnull[0m,
        blockNumber: [1mnull[0m,
        from: [32m"0x2e49e21e708b7d83746ec676a4afda47f1a0d693"[0m,
        gas: [31m90000[0m,
        gasPrice: [31m20000000000[0m,
        hash: [32m"0x7ccd5dc3a6498394320201fe09f9553cd5ce4aaa957363a6d2baca4be8309bd9"[0m,
        input: [32m"0x6060604052346000575b60458060156000396000f3606060405260e060020a6000350463c6888fa18114601c575b6000565b346000576029600435603b565b60408051918252519081900360200190f35b600781025b91905056"[0m,
        nonce: [31m59[0m,
        r: [32m"0xb35022f42f760331f2d2e0bb5530d05948da599b3b03510c2007a86eecf147b9"[0m,
        s: [32m"0x484e7f295f9999dd20da97c569c99bd34f0023260677ff46a2662c7ef9c8dcce"[0m,
        to: [1mnull[0m,
        transactionIndex: [1mnull[0m,
        v: [32m"0x1c"[0m,
        value: [31m0[0m
    }]


* 위 transactionHash를 가지고 transactionReceipt()를 실행하여 주소를 사용한다.
    * 주소는 마이닝 후 생성된다. 마이닝하지 않으면 주소는 null
    * transactionHash
    "0x7ccd5dc3a6498394320201fe09f9553cd5ce4aaa957363a6d2baca4be8309bd9"
    * contractAddress: "0x13581bb3c23492b722f230e967a0232741ccd247"
    * 주소의 bin code도 확인할 수 있다.
 


```python
!geth --exec 'eth.getTransactionReceipt("0x7ccd5dc3a6498394320201fe09f9553cd5ce4aaa957363a6d2baca4be8309bd9");' attach http://117.xxx.xxx.xxx:8xxx
```

    {
      blockHash: [32m"0x62c6f4ea5b26f1efbbbfa78dac40b29a6f8a9c3c14510b98301e8c16056a060e"[0m,
      blockNumber: [31m61711[0m,
      contractAddress: [32m"0x13581bb3c23492b722f230e967a0232741ccd247"[0m,
      cumulativeGasUsed: [31m40597[0m,
      from: [32m"0x2e49e21e708b7d83746ec676a4afda47f1a0d693"[0m,
      gasUsed: [31m40597[0m,
      logs: [],
      logsBloom: [32m"0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"[0m,
      root: [32m"0xdf63ca69b3ac58bf423229d501dff8897306e2b0c9a821c2fa6b4f19c980c98c"[0m,
      to: [1mnull[0m,
      transactionHash: [32m"0x7ccd5dc3a6498394320201fe09f9553cd5ce4aaa957363a6d2baca4be8309bd9"[0m,
      transactionIndex: [31m0[0m
    }


## E.8 이벤트

* 이벤트는 EVM 로그 기능을 사용한다.

* 함수에 listener를 설정할 수 있다.

단계 | 설명
-------|-------
개발-컴파일 | event넣은 소스를 컴파일해서 abi, bin을 구한다.
거래-배포 | 마이닝하고 주소를 배정받기 기다린다.
사용 | 이벤트 호출, _instance.MyLog()
사용 | 이벤트를 가지고 있는 함수 호출, _instance.MyFunction()

* 오류
    * "invalid adress error" default address가 설정되지 않으면, 누가 이벤트를 호출하는지 알 수 없다.
    ```
    > web3.eth.defaultAccount=web3.eth.accounts[0];
    ```

## 문제 e-12: contract 이벤트 사용하기 (nodejs)

* nodejs, Web3 라이브러리를 사용한다. 
* 이벤트가 있는 e_testDeposit.sol (컴파일은 geth)

* 오류
    * TypeError: contract.abi.filter is not a function
        * node에서 JSON.parse를 해줌.
    * Error: invalid address
        * web3.eth.defaultAccount를 설정.

### 1-3 e_testDeposit.sol

* 출처 인터넷
* deposit() 인자없이 호출해도 event 실행됨.


```python
%%writefile src/e_testDeposit.sol
pragma solidity ^0.4.0;

contract e_testDeposit {
    event Deposit(
        address indexed _from,
        bytes32 indexed _id,
        uint _value
    );
    function deposit(bytes32 _id) {
        Deposit(msg.sender, _id, msg.value);
    }
}
```

    Writing src/e_testDeposit.sol



```python
!solc --abi src/e_testDeposit.sol
```

    
    ======= e_testDeposit =======
    Contract JSON ABI
    [{"constant":false,"inputs":[{"name":"_id","type":"bytes32"}],"name":"deposit","outputs":[],"payable":false,"type":"function"},{"anonymous":false,"inputs":[{"indexed":true,"name":"_from","type":"address"},{"indexed":true,"name":"_id","type":"bytes32"},{"indexed":false,"name":"_value","type":"uint256"}],"name":"Deposit","type":"event"}]



```python
!solc --bin src/e_testDeposit.sol
```

    
    ======= e_testDeposit =======
    Binary: 
    606060405234610000575b60a7806100176000396000f360606040526000357c010000000000000000000000000000000000000000000000000000000090048063b214faa5146036575b6000565b34600057604e60048080359060200190919050506050565b005b80600019163373ffffffffffffffffffffffffffffffffffffffff167f19dacbf83c5de6658e14cbf7bcae5c15eca2eedecf1c66fbca928e4d351bea0f346040518082815260200191505060405180910390a35b5056


* 위 abi, bin을 복사해서 사용
* 아래는 eth를 사용해서 transactionHash, contractAddress 가져온다.
    * transactionHash:
"0xdd9182988b54e6e5587295a3cf3c82567ac99050804b6353918055d5d22b95fc"
    * contractAddress:
"0xf957f93a7b95c28005af20b5f8f396ba8b6f18d0"



```python
%%writefile src/e_testDeposit1.js
var _abiStr='[{"constant":false,"inputs":[{"name":"_id","type":"bytes32"}],"name":"deposit","outputs":[],"payable":false,"type":"function"},{"anonymous":false,"inputs":[{"indexed":true,"name":"_from","type":"address"},{"indexed":true,"name":"_id","type":"bytes32"},{"indexed":false,"name":"_value","type":"uint256"}],"name":"Deposit","type":"event"}]'
var _abiArray=JSON.parse(_abiStr);
var _bin="606060405234610000575b60a7806100176000396000f360606040526000357c010000000000000000000000000000000000000000000000000000000090048063b214faa5146036575b6000565b34600057604e60048080359060200190919050506050565b005b80600019163373ffffffffffffffffffffffffffffffffffffffff167f19dacbf83c5de6658e14cbf7bcae5c15eca2eedecf1c66fbca928e4d351bea0f346040518082815260200191505060405180910390a35b5056";
var _contract = eth.contract(_abiArray);
var _instance=_contract.new({data:_bin,from:web3.eth.accounts[0],gas:100000}, function(err, contract) {
  if (!err && contract.address)
    console.log("contractAddress: ", contract.address);
    console.log("transactionHash: ", contract.transactionHash);
});
console.log("after async - transactionHash: ",_instance.transactionHash)
console.log("after async - contractAddress: ",_instance.address);
```

    Writing src/e_testDeposit1.js



```python
!geth --exec 'loadScript("src/e_testDeposit1.js")' attach http://117.xxx.xxx.xxx:8xxx
```

    transactionHash:  0xdd9182988b54e6e5587295a3cf3c82567ac99050804b6353918055d5d22b95fc
    after async - transactionHash:  0xdd9182988b54e6e5587295a3cf3c82567ac99050804b6353918055d5d22b95fc
    after async - contractAddress:  undefined
    [1mtrue[0m



```python
!geth --exec 'eth.getTransactionReceipt("0xdd9182988b54e6e5587295a3cf3c82567ac99050804b6353918055d5d22b95fc");' attach http://117.xxx.xxx.xxx:8xxx
```

    {
      blockHash: [32m"0x9cdd5bf9ecacc957e34822169656362460681e3188b50eaf0c411d9c7ee87a7a"[0m,
      blockNumber: [31m64973[0m,
      contractAddress: [32m"0xf957f93a7b95c28005af20b5f8f396ba8b6f18d0"[0m,
      cumulativeGasUsed: [31m64967[0m,
      from: [32m"0x2e49e21e708b7d83746ec676a4afda47f1a0d693"[0m,
      gasUsed: [31m64967[0m,
      logs: [],
      logsBloom: [32m"0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"[0m,
      root: [32m"0x080fdbf33c6060a007ab43c896cdff27ff5f9da25a6518b29c77d4e73316c6e1"[0m,
      to: [1mnull[0m,
      transactionHash: [32m"0xdd9182988b54e6e5587295a3cf3c82567ac99050804b6353918055d5d22b95fc"[0m,
      transactionIndex: [31m0[0m
    }



```python
%%writefile src/e_testDeposit2.js
var Web3=require('web3');
var web3;
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    web3 = new Web3(new Web3.providers.HttpProvider("http://117.xxx.xxx.xxx:8xxx"));
}
console.log(web3.eth.blockNumber);
var _abiStr='[{"constant":false,"inputs":[{"name":"_id","type":"bytes32"}],"name":"deposit","outputs":[],"payable":false,"type":"function"},{"anonymous":false,"inputs":[{"indexed":true,"name":"_from","type":"address"},{"indexed":true,"name":"_id","type":"bytes32"},{"indexed":false,"name":"_value","type":"uint256"}],"name":"Deposit","type":"event"}]'
var _abiArray=JSON.parse(_abiStr);
var _address = '0xf957f93a7b95c28005af20b5f8f396ba8b6f18d0';
var _contract=web3.eth.contract(_abiArray);
var _instance=_contract.at(_address);
web3.eth.defaultAccount=web3.eth.accounts[0];
var event = _instance.Deposit(function (error, result) {
    if (!error) {
        console.log("event triggered ===> ",result);
        process.exit(1);
    }
});
_instance.deposit();

```

    Writing src/e_testDeposit2.js



```python
!node src/e_testDeposit2.js
```

    64982
    event triggered ===>  { address: '0xf957f93a7b95c28005af20b5f8f396ba8b6f18d0',
      blockHash: '0xbf9a68c71f2ff475f77d075857cc4592b36a520f966b43bf9bf188bb2fa82381',
      blockNumber: 64984,
      logIndex: 0,
      removed: false,
      transactionHash: '0xc053f34a73043e0e2c1ae79b8e2632b8b016d6b85834afd7d1e48497daf3d5f4',
      transactionIndex: 0,
      event: 'Deposit',
      args: 
       { _from: '0x2e49e21e708b7d83746ec676a4afda47f1a0d693',
         _id: '0x0000000000000000000000000000000000000000000000000000000000000000',
         _value: { [String: '0'] s: 1, e: 0, c: [Object] } } }


## 문제 e-13: 웹으로 contract 배포하고 사용하기

* 웹에서는 상대경로를 주의해야 한다.
    * html페이지를 기준으로 상대경로로 라이브러리를 읽는다.
    * 예: contract.html에서 web3.js는 '../node_modules/web3/dist/web3.js'
    ```
    projectDir
        |-ethereum.ipynb ---> 지금 사용하는 노트북
        |-node_modules/web3/dist/web3.js ---> 사용하는 라이브러리
        |-_script/contract.html ---> 여는 웹페이지
    ```

* 1-1 라이브러리 설치
* 1-2 Notebook 셀에서 사용하기
* 1-3 html에서 contract 컴파일 배포
* 1-4 html에서 계정잔고 변경 모니터링

### 1-1 라이브러리 설치 

* npm web3 설치
    ```
    $ cd ~/Code/git/bb/jsl/bitcoin/
    $ npm install web3
    ```



### 1-3 html에서 컴파일 배포

* 단계1: contract.html 작성
    * web3.js/example/contract.html을 수정해서 사용

항목 | 설명
-----|-----
web3.js | contract.html을 기준으로 상대경로 적음. src="..node_modules/web3/dist/web3.js"
HttpProvider | http://117.xxx.xxx.xxx:8xxx


* 단계 2: 브라우저에서 연다. 주의: web3.js 라이브러리는 contract.html 기준으로 상대경로.
    ```
    file:///projectDir/_scripts/contract.html
    ```
    * 버튼을 누르면 거래가 생성되고 대기한다. mining을 해서 처리해야 한다.
    * multiply 수를 넣는 화면이 뜨고, Enter를 누르면 계산 결과를 볼 수 있다.


```python
%%writefile _scripts/contract.html
<!doctype>
<html>

<head>
<script type="text/javascript" src="../node_modules/web3/dist/web3.js"></script>
<script type="text/javascript">
    var Web3 = require('web3');
    var web3 = new Web3(new Web3.providers.HttpProvider("http://117.xxx.xxx.xxx:8xxx"));

    // solidity code code
    var source = "" +
    "contract test {\n" +
    "   function multiply(uint a) constant returns(uint d) {\n" +
    "       return a * 7;\n" +
    "   }\n" +
    "}\n";

    var compiled = web3.eth.compile.solidity(source);
    var code = compiled.test.code;
    // contract json abi, this is autogenerated using solc CLI
    var abi = compiled.test.info.abiDefinition;

    var myContract;

    function createExampleContract() {
        // hide create button
        document.getElementById('create').style.visibility = 'hidden'; 
        document.getElementById('code').innerText = code;

        // let's assume that coinbase is our account
        web3.eth.defaultAccount = web3.eth.coinbase;
        alert(code);

        // create contract
        document.getElementById('status').innerText = "transaction sent, waiting for confirmation";
        web3.eth.contract(abi).new({data: code}, function (err, contract) {
            if(err) {
                console.error(err);
                return;
            // callback fires twice, we only want the second call when the contract is deployed
            } else if(contract.address){
                myContract = contract;
                console.log('address: ' + myContract.address);
                document.getElementById('status').innerText = 'Mined!';
                document.getElementById('call').style.visibility = 'visible';
            }
        });
    }
    function callExampleContract() {
        // this should be generated by ethereum
        var param = parseInt(document.getElementById('value').value);
        // call the contract
        var res = myContract.multiply(param);
        document.getElementById('result').innerText = res.toString(10);
    }
</script>
</head>
<body>
    <h1>contract</h1>
    <div id="code"></div> 
    <div id="status"></div>
    <div id='create'>
        <button type="button" onClick="createExampleContract();">!create example contract</button>
    </div>
    <div id='call' style='visibility: hidden;'>
        <input type="number" id="value" onkeyup='callExampleContract()'></input>
    </div>
    <div id="result"></div>
</body>
</html>


```

    Overwriting _scripts/contract.html



```python
import webbrowser
myurl='_scripts/contract.html'
webbrowser.open(myurl,new=2)
```




    True



### 1-4 html에서 계정잔고 변경 모니터링

* 단계1: balance.html 작성
    * web3.js/example/balance.html을 수정해서 사용
    * 계정의 금액에 변동이 있는지 filter

항목 | 설명
-----|-----
web3.js | balance.html을 기준으로 상대경로 적음. src="..node_modules/web3/dist/web3.js"
HttpProvider | http://117.xxx.xxx.xxx:8xxx


* 단계 2: 브라우저에서 연다. 주의: web3.js 라이브러리는 balance.html 기준으로 상대경로.
    ```
    file:///projectDir/_scripts/balance.html
    ```
    * 버튼을 누르고, 잔고변경이 있으면 결과를 볼 수 있다. mining은 필요 없다.
        ```
        coinbase: 0x2e49e21e708b7d83746ec676a4afda47f1a0d693
        original balance: 2.38565e+23 watching...
        current: 2.38575e+23
        diff: 10000000000001573000
        ```


```python
%%writefile _scripts/balance.html
<!doctype>
<html>

<head>
<script type="text/javascript" src="../node_modules/bignumber.js/bignumber.min.js"></script>
<script type="text/javascript" src="../node_modules/web3/dist/web3.js"></script>
<script type="text/javascript">
   
    var Web3 = require('web3');
    var web3 = new Web3(new Web3.providers.HttpProvider("http://117.xxx.xxx.xxx:8xxx"));

    alert(web3.isConnected());

    function watchBalance() {
        var coinbase = web3.eth.coinbase;
        alert(coinbase);
        var originalBalance = web3.eth.getBalance(coinbase).toNumber();
        alert(originalBalance);
        document.getElementById('coinbase').innerText = 'coinbase: ' + coinbase;
        document.getElementById('original').innerText = ' original balance: ' + originalBalance + '    watching...';

        web3.eth.filter('latest').watch(function() {
            var currentBalance = web3.eth.getBalance(coinbase).toNumber();
            document.getElementById("current").innerText = 'current: ' + currentBalance;
            document.getElementById("diff").innerText = 'diff:    ' + (currentBalance - originalBalance);
        });
    }

</script>
</head>
<body>
    <h1>---coinbase balance</h1>
    <button type="button" onClick="watchBalance();">watch balance</button>
    <div></div>
    <div id="coinbase"></div>
    <div id="original"></div>
    <div id="current"></div>
    <div id="diff"></div>
</body>
</html>


```

    Overwriting _scripts/balance.html


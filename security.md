## Overview : 👉👉👉
```bash
** While blockchain technology is gaining traction, there are potential attacks. For instance, there are currently
emerging DeFi attacks; exchange hacks, among others. However, all blockchain-related attacks are not smart contract attacks.
Some are defraud, malicious, and weak protocol attacks.

The Attack Classification ::📑📑📑
The attacks on smart contract security and blockchain, in general, can be classified into four basic categories.
These categories includes :-

1. malicious attacks: Crypto-jacking, slack, and forum attack

2. weak protocol :  the different protocols are prone to one attack or the other. Some of those attacks are 51% attacks, 
                  Sybill attacks, 34% attack, and denial of service.
                  An Eclipse attack can occur in the same vein to manipulate the peer-to-peer (P2P) network.
                  
3. defraud : Defraud can trick a merchant into releasing his goods before the confirmation of a transaction. 
             In a practical sense, a Bitcoin transaction is confirmed after six transactions.a consumer may try to persuade a merchant to 
             release goods without waiting for up to 6 transactions, so attack techniques like one confirmation or no confirmation 
             can be initiated to double spend.

4. application bugs :  It arises when smart contract developers fail to see code errors in the decentralized application. 
                        An attacker can drain all the money from the smart contract wallet through simple code bugs. Hence, the need 
                        for smart contract audits.
```
 Let's Explore different Security risks ===>> 
 
## 1. Reentrancy 🚩🚩🚩
```bash 
** info : 
While the EVM cannot run multiple contracts at the same time, a contract calling a different contract
pauses the calling contract's execution and memory state until the call returns, 
at which point execution proceeds normally. This pausing and re-starting can create a vulnerability known as "re-entrancy".

👉👉👉: The external calls can be hijacked by attackers, who can force the contracts to execute further 
code (through a fallback function), including calls back into themselves. Attacks of this kind were used in the infamous DAO hack.

** example :
contract Victim {
    mapping (address => uint256) public balances;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() external {
        uint256 amount = balances[msg.sender];
        (bool success, ) = msg.sender.call.value(amount)("");
        require(success);
        balances[msg.sender] = 0;
    }
}
------------------------------------------------------------------------------------------
** Imagine this malicious contract from attackers :
contract Attacker {
    function beginAttack() external payable {
        Victim(VICTIM_ADDRESS).deposit.value(1 ether)();
        Victim(VICTIM_ADDRESS).withdraw();
    }

    function() external payable {
        if (gasleft() > 40000) {
            Victim(VICTIM_ADDRESS).withdraw();
        }
    }
}
---------------------------------------------------------------------------------------------
** Calling Attacker.beginAttack() will start a cycle that looks something like :

0.) Attacker's EOA calls Attacker.beginAttack() with 1 ETH
0.) Attacker.beginAttack() deposits 1 ETH into Victim
1.) Attacker -> Victim.withdraw()
  1.) Victim reads balances[msg.sender]
  1.) Victim sends ETH to Attacker (which executes default function)
    2.) Attacker -> Victim.withdraw()
    2.) Victim reads balances[msg.sender]
    2.) Victim sends ETH to Attacker (which executes default function)
      3.) Attacker -> Victim.withdraw()
      3.) Victim reads balances[msg.sender]
      3.) Victim sends ETH to Attacker (which executes default function)
        4.) Attacker no longer has enough gas, returns without calling again
      3.) balances[msg.sender] = 0;
    2.) balances[msg.sender] = 0; (it was already 0)
  1.) balances[msg.sender] = 0; (it was already 0)
---------------------------------------------------------------------------------------------------------------->>>
Calling Attacker.beginAttack with 1 ETH will re-entrancy attack Victim, withdrawing more ETH 
than it provided (taken from other users' balances, causing the Victim contract to become under-collateralized.
---------------------------------------------------------------------------------------------------------------->>>

** Prevention : 
By simply switching the order of the storage update and external call, 
we prevent the re-entrancy condition that enabled the attack. Calling back into withdraw,
while possible, will not benefit the attacker, since the balances storage will already be set to 0.
----------------------->>>>>>>>>>>>
contract NoLongerAVictim {
    function withdraw() external {
        uint256 amount = balances[msg.sender];
        balances[msg.sender] = 0;
        (bool success, ) = msg.sender.call.value(amount)("");
        require(success);
    }
}
The code above follows the "Checks-Effects-Interactions" design pattern, which helps protect against re-entrancy.
---------------------------------------------------------------------------------------------------------------------
Method 2 : 
Any time you are sending ETH to an untrusted address or interacting with an unknown contract (such as calling transfer() 
of a user-provided token address), you open yourself up to the possibility of re-entrancy. 
By designing contracts that neither send ETH nor call untrusted contracts, you prevent the possibility of re-entrancy!
```
## 2. Arithmatic over/underflows 🚩🚩🚩
```bash 
** info : 
EVM specifies fixed-size data types for integers. This means that an integer variable can represent
only a certain range of numbers. A uint8, for example, can only store numbers in the range [0,255]. Trying to store 256 into a uint8 will result in 0. 
==>>  If care is not taken, variables in Solidity can be exploited if user input is unchecked and calculations are performed that result
in numbers that lie outside the range of the data type that stores them. 

** example: 
===>> For example, subtracting 1 from a uint8 (unsigned integer of 8 bits; i.e., nonnegative) variable whose value is 0
      will result in the number 255. This is an underflow.
====>>Adding numbers larger than the data type’s range is called an overflow. For clarity,
       adding 257 to a uint8 that currently has a value of 0 will result in the number 1. 
 ---------------------------CODING--------------------------------------------------------------------
 contract TimeLock {

    mapping(address => uint) public balances;
    mapping(address => uint) public lockTime;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
        lockTime[msg.sender] = now + 1 weeks;
    }

    function increaseLockTime(uint _secondsToIncrease) public {
        lockTime[msg.sender] += _secondsToIncrease;
    }

    function withdraw() public {
        require(balances[msg.sender] > 0);
        require(now > lockTime[msg.sender]);
        balances[msg.sender] = 0;
        msg.sender.transfer(balance);
    }
} 


NOW, Here users can deposit ether into the contract and it will be locked there for at least a week.
The user may extend the wait time to longer than 1 week if they choose, but once deposited, 
the user can be sure their ether is locked in safely for at least a week—or so this contract intends.

Note ====>> The attacker could determine the current lockTime for the address they now hold the key for (it’s a public variable). 
    Let’s call this userLockTime. They could then call the increaseLockTime function and pass as an argument the number 2^256 - userLockTime. 
    This number would be added to the current userLockTime and cause an overflow, resetting lockTime[msg.sender] to 0.
    The attacker could then simply call the withdraw function to obtain their reward
** Prevention :-
technique to guard against under/overflow vulnerabilities is to use or build mathematical libraries that replace 
the standard math operators addition, subtraction, and multiplication. 
===>> division does not cause over/underflows 
OpenZeppelin has done a great job of building and auditing secure libraries for the Ethereum community. 
In particular, its SafeMath library can be used to avoid under/overflow vulnerabilities.

--------------------  Coding ----------------------------------------------
library SafeMath {
 function mul(uint256 a, uint256 b) internal pure returns (uint256) {
    if (a == 0) {
      return 0;
    }
    uint256 c = a * b;
    assert(c / a == b);
    return c;
  }

  function div(uint256 a, uint256 b) internal pure returns (uint256) {
    // assert(b > 0); // Solidity automatically throws when dividing by 0
    uint256 c = a / b;
    // assert(a == b * c + a % b); // This holds in all cases
    return c;
  }

  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    assert(b <= a);
    return a - b;
  }

  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    assert(c >= a);
    return c;
  }
}

contract TimeLock {
    using SafeMath for uint; // use the library for uint type
    mapping(address => uint256) public balances;
    mapping(address => uint256) public lockTime;

    function deposit() external payable {
        balances[msg.sender] = balances[msg.sender].add(msg.value);
        lockTime[msg.sender] = now.add(1 weeks);
    }

    function increaseLockTime(uint256 _secondsToIncrease) public {
        lockTime[msg.sender] = lockTime[msg.sender].add(_secondsToIncrease);
    }

    function withdraw() public {
        require(balances[msg.sender] > 0);
        require(now > lockTime[msg.sender]);
        uint256 transferValue = balances[msg.sender];
        balances[msg.sender] = 0;
        msg.sender.transfer(transferValue);
    }
}
--------------------------------------------------------------------------------------------------------------------------------------------------
```
## 3. Unexpected ethers  🚩🚩🚩
```bash 
** info : 
 when ether is sent to a contract it must execute either the fallback function or another function defined in the contract.
 There are two exceptions to this, where ether can exist in a contract without having executed any code. 
 Contracts that rely on code execution for all ether sent to them can be vulnerable to attacks where ether is forcibly sent.
 ** example : 
** Prevention : 
```
## 4. Delegatecall 🚩🚩🚩
```bash 
** info : 
** example: 
** Prevention : 
```
## 5. Default Visibilities 🚩🚩🚩
```bash 
** info : 
-->>  incorrect use of visibility specifiers can lead to some devastating vulnerabilities in smart contracts. 
The default visibility for functions is public, so functions that do not specify their visibility will be callable by external users.
The issue arises when developers mistakenly omit visibility specifiers on functions that should be private.



** example: 

contract HashForEther {

    function withdrawWinnings() {
        // Winner if the last 8 hex characters of the address are 0
        require(uint32(msg.sender) == 0);
        _sendWinnings();
     }

     function _sendWinnings() {
         msg.sender.transfer(this.balance);
     }
}

* Note 1 : withdrawWinnings function can be called by any if they  generate an Ethereum address whose last 8 hex characters are 0. 
the _sendWinnings function is also public (the default), and thus any address can call this function to steal the bounty. 
* Note 2 :nowadays versions of solc show a warning for functions that have no explicit visibility set, to encourage this practice.

example 2 :-->>
----------------------- by this contract about $31M worth of Ether was stolen ----------------

contract WalletLibrary is WalletEvents {

  // constructor is given number of sigs required to do protected
  // "onlymanyowners" transactionsas well as the selection of addresses
  // capable of confirming them
  function initMultiowned(address[] _owners, uint _required) {
    m_numOwners = _owners.length + 1;
    m_owners[1] = uint(msg.sender);
    m_ownerIndex[uint(msg.sender)] = 1;
    for (uint i = 0; i < _owners.length; ++i)
    {
      m_owners[2 + i] = uint(_owners[i]);
      m_ownerIndex[uint(_owners[i])] = 2 + i;
    }
    m_required = _required;
  }

  ...

  // constructor - just pass on the owner array to multiowned and
  // the limit to daylimit
  function initWallet(address[] _owners, uint _required, uint _daylimit) {
    initDaylimit(_daylimit);
    initMultiowned(_owners, _required);
  }
}

** Prevention :
so, the function visibility must be declared carefully. 
====>>  Private functions must not be declared public. 


```
## 6. Entropy Illusion 🚩🚩🚩
```bash 
** info : 
** example: 
** Prevention : 
```
## 7. External Contract Referencing 🚩🚩🚩
```bash 
** info : 
** example: 
** Prevention : 
```
## 8. Short Address/Parameters Attack 🚩🚩🚩 
```bash 
** info : 
is when a contract receives less data than it was expecting, and Solidity fills the missing bytes with zeros. 
** example: 
contract NonPayloadAttackableToken {
  modifier onlyPayloadSize(uint size) {
    assert(msg.data.length == size + 4);
    _;
  } 

 function transfer(address _to, uint256 _value) onlyPayloadSize(2 * 32) {
   // do stuff
 }   
} 
-----------------------------------------
The transfer function expects two parameters. Each parameter has 32 bytes and the method signature adds 4 bytes to the call.
The expected total size is 68 bytes.
it is found that the EVM pads call from this multisignature wallet, making the total 96 bytes instead of the expected 68.
This multisignature wallet executes transactions in two steps :--->>

An operation is proposed
It is confirmed and executed
A simplified multisignature wallet looks like this:
function propose(address _to, uint _value, bytes _data) public returns (bool _success) {
   to = _to;
   value = _value;
   data = _data;

   return true;
}

function confirm() returns (bool _success) {
   if (!to.call.value(value)(data)) {
     throw;
   }

   return true;
}
-------------------------------------------------------------
In the proposed method, _data is initially formatted correctly to 68 bytes.

When the final call is made in confirm, it will send 68 bytes to our contract, but Solidity will pad it to 96 bytes. 
** Prevention : 
```
## 9. Unchecked Call Return Values 🚩🚩🚩
```bash 
** info : 
** example: 
** Prevention : 
```
## 10. Race conditions/front running 🚩🚩🚩
```bash
** info : 
** example: 
** Prevention : 
```
## 11. DDOS attack 🚩🚩🚩
```bash 
** info : 
** example: 
** Prevention : 
```
## 12. Block-timestamp Manipulation 🚩🚩🚩
```bash 
** info : 
** example: 
** Prevention : 
```
## 13. Constructors with Care 🚩🚩🚩
```bash 
** info : 
** example: 
** Prevention : 
```
## 14. Uninitialized Storage Pointers 🚩🚩🚩
```bash 
** info : 
** example: 
** Prevention : 
```
## 15. Floating Point and Precision 🚩🚩🚩
```bash 
** info : 
** example: 
** Prevention : 
```
## 16. Tx.Origin Authentication 🚩🚩🚩
```bash 
```
## 17. Contract Libraries 🚩🚩🚩
```bash 
** info : 
** example: 
** Prevention : 
```





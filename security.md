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
** example: 
** Prevention : 
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
so, the function visibility must be declared carefully. 


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
** example: 
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





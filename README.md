# Hacking Smart Contracts

## Code
```
/* This program is free software. It comes without any warranty, to
the extent permitted by applicable law. You can redistribute it
and/or modify it under the terms of the Do What The Fuck You Want
To Public License, Version 2, as published by Sam Hocevar. See
http://www.wtfpl.net/ for more details. */

/* These contracts are examples of contracts with vulnerabilities in order to practice your hacking skills.
DO NOT USE THEM OR GET INSPIRATION FROM THEM TO MAKE CODE USED IN PRODUCTION 
You are required to find vulnerabilities where an attacker harms someone else.
Being able to destroy your own stuff is not a vulnerability and should be dealt at the interface level.
*/

pragma solidity ^0.4.10;
//*** Exercice 1 ***//
// Simple token you can buy and send.
contract SimpleToken{
    mapping(address => uint) public balances;
    
    /// @dev Buy token at the price of 1ETH/token.
    function buyToken() payable {
        balances[msg.sender]+=msg.value / 1 ether;
    }
    
    /** @dev Send token.
     *  @param _recipient The recipient.
     *  @param _amount The amount to send.
     */
    function sendToken(address _recipient, uint _amount) {
        require(balances[msg.sender]!=0); // You must have some tokens.
        
        balances[msg.sender]-=_amount;
        balances[_recipient]+=_amount;
    }
    
}

//*** Exercice 2 ***//
// You can buy voting rights by sending ether to the contract.
// You can vote for the value of your choice.
contract VoteTwoChoices{
    mapping(address => uint) public votingRights;
    mapping(address => uint) public votesCast;
    mapping(bytes32 => uint) public votesReceived;
    
    /// @dev Get 1 voting right per ETH sent.
    function buyVotingRights() payable {
        votingRights[msg.sender]+=msg.value/(1 ether);
    }
    
    /** @dev Vote with nbVotes for a proposition.
     *  @param _nbVotes The number of votes to cast.
     *  @param _proposition The proposition to vote for.
     */
    function vote(uint _nbVotes, bytes32 _proposition) {
        require(_nbVotes + votesCast[msg.sender]<=votingRights[msg.sender]); // Check you have enough voting rights.
        
        votesCast[msg.sender]+=_nbVotes;
        votesReceived[_proposition]+=_nbVotes;
    }

}

//*** Exercice 3 ***//
// You can buy tokens.
// The owner can set the price.
contract BuyToken {
    mapping(address => uint) public balances;
    uint public price=1;
    address public owner=msg.sender;
    
    /** @dev Buy tokens.
     *  @param _amount The amount to buy.
     *  @param _price  The price to buy those in ETH.
     */
    function buyToken(uint _amount, uint _price) payable {
        require(_price>=price); // The price is at least the current price.
        require(_price * _amount * 1 ether <= msg.value); // You have paid at least the total price.
        balances[msg.sender]+=_amount;
    }
    
    /** @dev Set the price, only the owner can do it.
     *  @param _price The new price.
     */
    function setPrice(uint _price) {
        require(msg.sender==owner);
        
        price=_price;
    }
}

//*** Exercice 4 ***//
// Contract to store and redeem money.
contract Store {
    struct Safe {
        address owner;
        uint amount;
    }
    
    Safe[] public safes;
    
    /// @dev Store some ETH.
    function store() payable {
        safes.push(Safe({owner: msg.sender, amount: msg.value}));
    }
    
    /// @dev Take back all the amount stored.
    function take() {
        for (uint i; i<safes.length; ++i) {
            Safe safe = safes[i];
            if (safe.owner==msg.sender && safe.amount!=0) {
                msg.sender.transfer(safe.amount);
                safe.amount=0;
            }
        }
        
    }
}

//*** Exercice 5 ***//
// Count the total contribution of each user.
// Assume that the one creating the contract contributed 1ETH.
contract CountContribution{
    mapping(address => uint) public contribution;
    uint public totalContributions;
    address owner=msg.sender;
    
    /// @dev Constructor, count a contribution of 1 ETH to the creator.
    function CountContribution() public {
        recordContribution(owner, 1 ether);
    }
    
    /// @dev Contribute and record the contribution.
    function contribute() public payable {
        recordContribution(msg.sender, msg.value);
    }
    
    /** @dev Record a contribution. To be called by CountContribution and contribute.
     *  @param _user The user who contributed.
     *  @param _amount The amount of the contribution.
     */
    function recordContribution(address _user, uint _amount) {
        contribution[_user]+=_amount;
        totalContributions+=_amount;
    }
    
}

//*** Exercice 6 ***//
contract Token {
    mapping(address => uint) public balances;
    
    /// @dev Buy token at the price of 1ETH/token.
    function buyToken() payable {
        balances[msg.sender]+=msg.value / 1 ether;
    }
    
    /** @dev Send token.
     *  @param _recipient The recipient.
     *  @param _amount The amount to send.
     */
    function sendToken(address _recipient, uint _amount) {
        require(balances[msg.sender]>=_amount); // You must have some tokens.
        
        balances[msg.sender]-=_amount;
        balances[_recipient]+=_amount;
    }
    
    /** @dev Send all tokens.
     *  @param _recipient The recipient.
     */
    function sendAllTokens(address _recipient) {
        balances[_recipient]=+balances[msg.sender];
        balances[msg.sender]=0;
    }
    
}

//*** Exercice 7 ***//
// You can buy some object.
// Further purchases are discounted.
// You need to pay basePrice / (1 + objectBought), where objectBought is the number of object you previously bought.
contract DiscountedBuy {
    uint public basePrice = 1 ether;
    mapping (address => uint) public objectBought;

    /// @dev Buy an object.
    function buy() payable {
        require(msg.value * (1 + objectBought[msg.sender]) == basePrice);
        objectBought[msg.sender]+=1;
    }
    
    /** @dev Return the price you'll need to pay.
     *  @return price The amount you need to pay in wei.
     */
    function price() constant returns(uint price) {
        return basePrice/(1 + objectBought[msg.sender]);
    }
    
}

//*** Exercice 8 ***//
// You choose Head or Tail and send 1 ETH.
// The next party send 1 ETH and try to guess what you chose.
// If it succeed it gets 2 ETH, else you get 2 ETH.
contract HeadOrTail {
    bool public chosen; // True if head/tail has been chosen.
    bool lastChoiceHead; // True if the choice is head.
    address public lastParty; // The last party who chose.
    
    /** @dev Must be sent 1 ETH.
     *  Choose head or tail to be guessed by the other player.
     *  @param _chooseHead True if head was chosen, false if tail was chosen.
     */
    function choose(bool _chooseHead) payable {
        require(!chosen);
        require(msg.value == 1 ether);
        
        chosen=true;
        lastChoiceHead=_chooseHead;
        lastParty=msg.sender;
    }
    
    
    function guess(bool _guessHead) payable {
        require(chosen);
        require(msg.value == 1 ether);
        
        if (_guessHead == lastChoiceHead)
            msg.sender.transfer(2 ether);
        else
            lastParty.transfer(2 ether);
            
        chosen=false;
    }
}

//*** Exercice 9 ***//
// You can store ETH in this contract and redeem them.
contract Vault {
    mapping(address => uint) public balances;

    /// @dev Store ETH in the contract.
    function store() payable {
        balances[msg.sender]+=msg.value;
    }
    
    /// @dev Redeem your ETH.
    function redeem() {
        msg.sender.call.value(balances[msg.sender])();
        balances[msg.sender]=0;
    }
}

//*** Exercice 10 ***//
// You choose Head or Tail and send 1 ETH.
// The next party send 1 ETH and try to guess what you chose.
// If it succeed it gets 2 ETH, else you get 2 ETH.
contract HeadTail {
    address public partyA;
    address public partyB;
    bytes32 public commitmentA;
    bool public chooseHeadB;
    uint public timeB;
    
    
    
    /** @dev Constructor, commit head or tail.
     *  @param _commitmentA is keccak256(chooseHead,randomNumber);
     */
    function HeadTail(bytes32 _commitmentA) payable {
        require(msg.value == 1 ether);
        
        commitmentA=_commitmentA;
        partyA=msg.sender;
    }
    
    /** @dev Guess the choice of party A.
     *  @param _chooseHead True if the guess is head, false otherwize.
     */
    function guess(bool _chooseHead) payable {
        require(msg.value == 1 ether);
        require(partyB==address(0));
        
        chooseHeadB=_chooseHead;
        timeB=now;
        partyB=msg.sender;
    }
    
    /** @dev Reveal the commited value and send ETH to the winner.
     *  @param _chooseHead True if head was chosen.
     *  @param _randomNumber The random number chosen to obfuscate the commitment.
     */
    function resolve(bool _chooseHead, uint _randomNumber) {
        require(msg.sender == partyA);
        require(keccak256(_chooseHead, _randomNumber) == commitmentA);
        require(this.balance >= 2 ether);
        
        if (_chooseHead == chooseHeadB)
            partyB.transfer(2 ether);
        else
            partyA.transfer(2 ether);
    }
    
    /** @dev Time out party A if it takes more than 1 day to reveal.
     *  Send ETH to party B.
     * */
    function timeOut() {
        require(now > timeB + 1 days);
        require(this.balance>=2 ether);
        partyB.transfer(2 ether);
    }
}

//*** Exercice 11 ***//
// You can create coffers put money into it and withdraw it.
contract Coffers {
    struct Coffer {uint[] slots;}
    mapping (address => Coffer) coffers;
    
    /** @dev Create coffers.
     *  @param _extraSlots The amount of slots to add to one's coffer.
     * */
    function createCoffers(uint _extraSlots) {
        Coffer coffer = coffers[msg.sender];
        require(coffer.slots.length+_extraSlots >= _extraSlots);
        coffer.slots.length += _extraSlots;
    }
    
    /** @dev Deposit money in one's coffer slot.
     *  @param _slot The slot to deposit money.
     * */
    function deposit(uint _slot) payable {
        Coffer coffer = coffers[msg.sender];
        coffer.slots[_slot] += msg.value;
    }
    
    /** @dev withdraw all of the money of  one's coffer slot.
     *  @param _slot The slot to withdraw money from.
     * */
    function withdraw(uint _slot) {
        Coffer coffer = coffers[msg.sender];
        msg.sender.transfer(coffer.slots[_slot]);
        coffer.slots[_slot] = 0;
    }
}
//*** Exercice Bonus ***//
// One of the previous contracts has 2 vulnerabilities.
// Find which one and describe the second vulnerability.
```

## Exercise Wise Solutions

### Exercise 1
Sender's balance should be more than amount also
`require(balances[msg.sender] >= _amount)`

Underflow won't occur now because of above condition

Overflow might occur
`require(balances[_recipient] <= balances[_recipient] + _amount)`

### Exercise 2
If _nbVotes is near 2^256-1 then `require(_nbVotes + votesCast[msg.sender]<=votingRights[msg.sender]);` may still be true due to overflow so we need more checks

Checking overflows
`require(votesCast[msg.sender] <= votesCast[msg.sender] + _nbVotes);`
`require(votesReceived[_proposition] <= votesReceived[_proposition] + _nbVotes);`

### Exercise 3
Avoiding overflow which can occur due to multiplication
`require(_amount * _price/_amount == _price);`
or
`require(_amount > 0)`
`require(_amount * _price/_price == _amount);`

### Exercise 4
Everytime someone wants to store something the contract will create a separate safe for every call(this can be improved fundamentally by using just a mapping between address and storeValue).
`mapping (address => uint) safes`

store would be like this
`require(safes[msg.sender] <= safes[msg.sender] += msg.value)`
`safes[msg.sender] += msg.value`

Still for this one,
Using loops of dynamic length is not safe. Contract can get stuck in a loop easily and will consume lot of gas. Also if someone wants to take all his money, he'll still have to wait for all the loop iterations and hence this is extremely costly method.

### Exercise 5
`recordContribution` should be private instead of default public to avoid anyone changing the record without paying. Also check statements should be present to avoid overflow.

### Exercise 6
Overflow check for `balances[_recipient]`
`require(balances[_recipient] <= balances[_recipient]+=_amount)`

In sendAllTokens there is a typo `balances[_recipient]=+balances[msg.sender]`, instead of `=+` it should be `+=` along with overflow check for the same.

### Exercise 7
`price() constant returns(uint price)` means it will return price variable without having return statement and just remove this variable.
Like this 
```
function price() constant returns(uint) {
        price basePrice/(1 + objectBought[msg.sender]);
    }
```
1/2 = 0.5 1/3 = 0.333333... so `require(msg.value * (1 + objectBought[msg.sender]) == basePrice);` will not be true in odd cases because user will send the money(msg.value) based upon price() and that will not be accurate always. 
Instead we can change this statement to `require(msg.value * (1 + objectBought[msg.sender]) >= basePrice);` and return the user the extra money like this `msg.sender.transfer(msg.value - price());`

### Exercise 8
The contract looks almost perfect but it has a severe bug, `lastChoiceHead` is a private variable by default still anyone can access it by calling the contract so the guesser can call this first and then based on it's value will send the call to contract.

So to avoid this we may use a notary who will collect values from both chooser and guesser and will then compare them and reward the winner.
If we don't want third party, we should use encryption somewhat similar to exercise 10.

### Exercise 9
This is an example of [re-entrancy](https://solidity.readthedocs.io/en/v0.5.8/security-considerations.html#re-entrancy). A contract can call function redeem and get multiple refunds because `balances[msg.sender]=0;` won't happen until the first transfer successfully happens.
"Ether transfer can always include code execution, so the recipient could be a contract that calls back into redeem. This would let it get multiple refunds and basically retrieve all the Ether in the contract"

The fix would be 
```
uint balance = balances[msg.sender];
balances[msg.sender]=0;
msg.sender.transfer(_balance);
```
Using transfer is better than call() as it only passes 2300 gas to the fallback function.

### Exercise 10
The contract is supposed to behave in a way so that anyone can guess but that won't happen.After someone has guessed the value, the next time anyone calls `guess()` it will fail on the statement `require(partyb == address(0)` because partyB already exists so no one will be able to guess affter the first party.
Fix would be setting `partyB=address(0)` when resolving and in timeout too, if we just remove the `require(partyb == address(0)` check then anyone can guess on top of someone else's guess. 
ALso choice of B is always visible `chooseHeadB` so A will know when he is losing hence he will probably never call resolve by himself and the contract will be pushed into unnecessary timeout always(not a vulnerability).

### Exercise 11
If a new user(A) is calling createCoffers then in the first step we will define a coffer struct like this
`Coffer coffer = coffers[msg.sender];`
Now since `coffers[msg.sender]` doesn't exist yet, it will be default initialized value 0. We know that in solidity, complex data types defaults to storage when initialized. So coffer will be a storage pointer, pointing to slot 0.

Suppose A deposits some money on his slots. Now a new user B if calls withdraw, the coffer created will point to storage slot of A to read the value because he won't find msg.sender in coffers. And hence B can steal funds of A in that case.

Deposit and CreateCoffer should do overflow check


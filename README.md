# BlockchainTD6
## Create Infura account
![image](https://user-images.githubusercontent.com/40137577/112385006-5f5dfe80-8cef-11eb-85ea-7d02c23340d9.png)

## Create ERC721 token contract
```
pragma solidity >=0.6.0 <0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "contracts/Ownable.sol";

contract NFTA is ERC721 {...}
```

## Implement a registerBreeder() function ( whitelisting)
```
mapping(address => bool) public whitelist;

function registerBreeder(address new_breeder) onlyOwner public returns(bool success) {
        require(msg.sender == owner);
        if (!whitelist[new_breeder]) {
            whitelist[new_breeder] = true;
            success = true; 
        }
    }
    
function isWhitelisted(address _address) public view returns(bool) {
        return whitelist[_address];
    }
    
```
## Implement a declareAnimal() function
```
struct Animal {
        string name;
        uint dna;
        string family;
        string color;

    }
mapping(uint256 => Animal) public animals;

function declareAnimal (string memory _name, uint _dna , string family, string color)public returns(uint newId) {
        require(isWhitelisted(msg.sender));
        
        token = Animal(_name,_dna,_family,_color);
        
        uint256 newItemId = _tokenIds.current();
        animals[newItemId] = token;
        _mint(msg.sender, newItemId);
        _tokenIds.increment();
        return newItemId;
    }
```
## Implement a deadAnimal() function

```
function deadAnimal(uint animalId)public returns(bool success){
        require(msg.sender == _tokenOwners[animalId]);
        burn(animalId);
        delete animals[animalId];

    }
```
## Implement a breedAnimal() function
```
function breedAnimal(uint animalId, uint targetId, string name, string color,string family)public returns (uint newId){
        require(msg.sender == _tokenOwners[animalId]);
        require(msg.sender == _tokenOwners[targetId]);
        animalDna = animals[animalId].dna();
        uint newDna = ( animals[animalId].dna()+ animals[targetId].dna()) / 2;
        uint256 newItemId = _tokenIds.current();
        animals[newId] = token;
        _mint(msg.sender, newItemId);
        animals[newItemId] = Animal(name,newDna,family,color);
        _tokenIds.increment();
        return newItemId;
    }
```
## Auction
```
   struct Auction {
        uint itemId;
        address seller;
        uint price;
        uint expirationDate = block.timestamp;
        address winner
    }
```
## createAuction 
```
function createAuction(uint animalId, uint price) public returns (bool success){
        require(msg.sender == _tokenOwners[animalId]);
        newAuction = Auction(animalId,msg.sender, price)
        auctions[animalId] = newAuction
        
    }
```
price is to set a minimum price for the auction


## bidOnAuction
```
function bidOnAuction(uint itemId, uint price) public returns (bool success){
        require(block.timestamp <= thisAuction.expirationDate + 2 days);
        require(price <= msg.sender.balance);
        require(price > auctions[itemId].price);
        auctions[itemId].price = price;
        auctions[itemId].winner = msg.sender;
    }
```
## claiAuction
```
function claimAuction(uint itemId) public payable returns (bool success){
        require(block.timestamp >= thisAuction.expirationDate + 2 days);
        require(auctions[itemId].winner == msg.sender);
     
    }
```
## Fights
```
struct Fight{
        uint256 id,
        uint256 bet,
        address winner
    }
    mapping(uint => Fight) public fights;
    Counters.Counter private fightId;

    function proposeToFight(uint animalId) public payable {
        require(msg.value != 0);
        require(msg.sender == _tokenOwners[animalId]);
        fights[animalId].value = msg.value;
        fightId.increment();
        fights[animalId].id = fightId.current();
    
    }

    function agreeToFight(uint myAnimalId, uint opponentId) public payable{
        require(msg.value != 0);
        require(msg.sender == _tokenOwners[myAnimalId]);
        require(fights[opponentId].value != 0);
        require (fights[opponentId].value == msg.value );
        fight(myAnimalId, opponentId);
    }
    function fight(uint firstId, uint secondId) public returns (bool success){
    if (randomNumber() % 2 == 0) {
           fights[firstId].winner = ownerOf(firstId);
        } else {
            fights[firstId].winner = ownerOf(secondId);
        }   
    }
    function randomNumber() internal view returns (uint) {
    return uint(blockhash(block.number - 1));
}
```

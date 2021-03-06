# CryptoPoker
This is a decentralized poker application that allows two users to play poker in any part of the world using the ethereum blockchain. This code was made as the midterm project for the Dapp course from theschool.ai

## How the game works

### PAckages needed:

The game has three main necesary components for its correct perfmormance:
  * Solidity contract.
  * Web3.js to comunicate with the contract.
  * Wnode(whisper node) for asymetric encrypted comunication.
  * IPFS for the storage of the mutimedia images.(./png/ipfs_links.txt)
  
### The game:

The game starts with whenever a player makes a request to the contract to `joinCreateGame`. This function indicates the contract that theres a user that wants to play cryptopoker. The contract will look if there are any available games for the player to join or, if not, create a new game for the new player.
#### Game Rules:
  * There can only be 2 players per game
  * A player can only participate in one game at once
  * The first player joining a game will be called Player1 while the 2nd will be called Player2

```solidity
function joinCreateGame() public{
        uint usrGame = playersGame[msg.sender];
        bool usrActive = gameActive[usrGame];
        require(usrActive != true);
        //check is there is a game open
        //if so, join the game
        if(gamePlayerCount[current_game]<2){
            playersGame[msg.sender]=current_game;
            playersInGame[current_game].player2 = msg.sender;
            gamePlayerCount[current_game]++;
            //trigger the event that a new game has started
            emit newGame(current_game);
        }
        else{
            current_game++;
            playersGame[msg.sender]=current_game;
            gamePlayerCount[current_game] = 1;
            gameActive[current_game] = true;
            playersInGame[current_game].player1 = msg.sender;
        }
    }
```

A gmae is considered ready to start once it has 2 players on it, at this point, the contract will trigger the event `newGame`. This trigger will indicate the web3.js interface that the game is ready to begin.

The first step once the game is ready is to retrieve the fellow games's address. This will be done by calling the contracts `getPartner` which is an external function restricted to players participating in active games and only to retrieve information about their own games. `getPartner` will return an address which will be use to begin comunication with the other player via a whisper node. the Wnode will create a conversation utilizing the `partnerAdres` as the public key for asymetric encryption, meaning that only the person with that public id will have acces to the conversation.

This newly created conversation will allow both of the players to interact with each other.

The cardimages are stored with IPFS to ensure inmutability of the cards. Each of the cards wil have its own  IPFS address and the only time a player will have acces to all of them will be at the beggining of the game.

	https://gateway.ipfs.io/ipfs/Qmeg8FeV6TTLQLiCh6tBTZtqHhgAGj3ELwNQMmFeTUsQAh

The player1 will start the game. we will use a tequnique based on additive ecryption in order to ensure the legality of the game.

### Additive encryption

Given a key _K1_ and a string _str_ the function for encryption will be given by:
      
      FK1(str) = encryptedK1_str
 
 The additive property makes it so that given a second key _K2_:
 
      
      FK2(FK1(str)) = FK1(FK2(str)) = encryptedK1K2_str = encryptedK2K1_str
      
 Solving for str:
 
      str = ((encryptedK1K2_str)K1^-1)K2^-1 = ((encryptedK1K2_str)K2^-1)K1^-1
 
 
### Dealing the cards

Given the 52 card addresse player1 will be asked to select a key `p1Key` to encrypt each one of them, shuffle the order of the cards, and then send the 52 encrypted links to player2 using the Wnode.

Once the player2 has received the cards, he will be ecouraged to shuffle them and then select  a key `P2Key` to encrypt all of the cards except 5 of them. he will then return the cards to player1.

For this second round, player1 will be able to see the 5 cards which player2 did not encrypt with his key.
Those 5 cards will be `players1Hand`. He will also have to remove his encryption from 5 of the remaining cards (he is highly encouraged to shuffle them first) and re send them to player2.

Player2 will now be able to see his 5 cards `player2Hand`.

### Bidding

Once Both players have their cards, Player1 will be asked to make a bid. if the bid is equal 0, the game will inmediatly end. Otherwise, the bid will be placed in the ethereum blockchain.

Afterwards, Player2 will be asked to make a bid equal or higher than Player1's bid. If he is not able to do it, the game will end imediatly.

If both players Placed a valid key, Player 1 will proced to submit his cards to the blockchain. and afterwards will be compared with player2 cards. For the sake of simplicity, the winer will be the player with more points where each card s worth its numer in points (e.g.g. 3spade,2heart,4club = 9points).

The player with the most points will be the winner and will be able to withdrawl the total bid.

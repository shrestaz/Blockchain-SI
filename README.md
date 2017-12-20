# Blockchain

## What is Blockchain?

Blockchain is a technology that allows for fast, secure and transparent peer-to-peer transfer of digital goods including money and intellectual property. It allows for those information to be distributed and not copied. It can also be defined as an incorruptible digital ledger of transactions.

* While being originally formulated for digitally currency, “Bitcoin”, there has evolved other potential uses of this technology. *

Blockchain holds the information as a shared and continually reconciled database. Since it is shared and hosted by millions of computers simultaneously, it is truly public and easily verifiable. There doesn’t exist a centralized version which can be hacked and/or tampered with.

To summarize, blockchain consists of

A network of nodes
Decentralization
Transparent and incorruptible
Distributed database

The use cases of this technology has spread like wildfire. It is used widely on online transactions. It is also implemented on day-to-day tools like GitHub and Google Docs, for example.

* Source: https://blockgeeks.com/guides/what-is-blockchain-technology/

## Requirements:

The following are the formal requirements for the project.
A peer-to-peer networking
A blockchain network consisting of 4 nodes.
An implementation with interactive API endpoint
Reproducible setup in a Linux based system with example nodes and test scenario
Demonstration of the running system
[A complete description provided for the requirements can be found here.](https://github.com/datsoftlyngby/soft2017fall-system-integration-teaching-material/blob/master/assignments/Blockchain.md)

## Implementation:

In simple terms, our approach consists of a Node.js implementation with Express framework. It is instantiated by Docker-Compose as nodes with a peer-to-peer networking.
It also uses HTTP interface to interact with a node and Websockets to communicate with other nodes as a peer-to-peer connection.

### Architecture


The nodes exposes two webservers.

For user interaction : HTTP server

Peer-to-peer connection between nodes: Websocket server

### Block Structure:

The structure of the block consists of:

Index: An auto incrementing index as new blocks are added.

Previous hash: Hash of the previous block to calculate new block’s hash. It ensure integrity when adding to the blockchain.

Time stamp: the current timestamp from collected from the host machine to log the new block.

Data: This data is the only manual input from the user. This can be any piece of information, the user wants to add to the blockchain.

Hash: This is the hash value is calculated after all the above variables has been fulfilled, using SHA256 cryptographic hash function.

Nonce: This is the Proof-of-Work part of the blockchain, calculated by the method `mineblock()`. This is calculated until the mined hash has a difficulty of level 4 (which can be adjusted as needed), meaning the first 4 values of the above mined hash equals to 0.

```class Block {
    constructor(index, previousHash, timestamp, data, hash) {
        this.index = index;
        this.previousHash = previousHash;
        this.timestamp = timestamp;
        this.data = data;
        this.hash = this.calculateHash();
        this.mineBlock();
    }
```





### Block Hash:

The following method mines the hash of the block with difficulty 4. This is to keep the integrity of the data.

 ```
    calculateHash() {
        return CryptoJS(this.index + this.previousHash + this.timestamp + JSON.stringify(this.data) + this.nonce).toString();
    }
  
    mineBlock(difficulty) {
        this.nonce = 0;
        while (this.hash.substring(0, 4) !== "0000") {
          this.nonce++;
          this.hash = this.calculateHash();
      }
```

### Storing the Blocks:
To store the blockchain, an in-memory Javascript array is used. The first block, also known as “Genesis block” is hardcoded.

```
var getGenesisBlock = () => {
    return new Block(0, "0", 1465154705, "I'm the first block. My friends call me Genesis block.", "0");
};

var blockchain = [getGenesisBlock()];

```

### Handling the new blocks:
All the nodes are synchronized with each other. Meaning, each nodes listens for changes in other nodes. When a new block is mined and added to the chain in one of the nodes, the other nodes checks for its integrity. Only if the new block’s index number is greater than the existing block in the node’s chain, then the new block get’s added.

### Nodes:
The user is able to interact with the nodes using an HTTP server.
```
var initHttpServer = () => {
    var app = express();
    app.use(bodyParser.json());

    app.get('/blocks', (req, res) => res.send(JSON.stringify(blockchain)));
    app.post('/mineBlock', (req, res) => {
        var newBlock = generateNextBlock(req.body.data);
        addBlock(newBlock);
        broadcast(responseLatestMsg());
        console.log('block added: ' + JSON.stringify(newBlock));
        res.send();
    });
    app.get('/peers', (req, res) => {
        res.send(sockets.map(s => s._socket.remoteAddress + ':' + s._socket.remotePort));
    });
    app.post('/addPeer', (req, res) => {
        connectToPeers([req.body.peer]);
        res.send();
    });
    app.listen(http_port, () => console.log('Listening http on port: ' + http_port));
};
```
The users can do the following with the nodes:

List all blocks: using `/blocks` route

Create and add new block on a node with desired data: using `/mineblock` route

List all peer: using `/peers` route

Add new peer: using `/addPeer` route

*Source: https://medium.com/@lhartikk/a-blockchain-in-200-lines-of-code-963cc1cc0e54



## Alternative methods of mining:

There has been a server created for returning a hash of difficulty 4. That means the mined hash will start with four 0s.

https://sleepy-cove-43230.herokuapp.com

![mother](https://github.com/expert26111/MotherProcessBlockhain/blob/master/herokurequest.png)
*Making a post request using PostMan* 

When making a post request, the block object is passed, mining process initiates and hash is returned.

** Alternate method 1: **

We can create a few servers like this, first calculating nonce starting from 0 to 2000, second calculating from 2000 to 4000 and so on. Dividing the process and initiating them in parallel would result in faster mining approach.

**Problem:** 

This requires advanced javascript code.

**Alternate method 2:**

Another way for mining different than while loop was using the child-process npm module. On a condition here, another node.js script can be started and the result of it can used. 

**Problem:**

The main problem with this approach is that on the message received from the child process, a global variable has to be changed.

[Link for the child process.](https://github.com/expert26111/ChildProcessCode/blob/master/mining1.js)

[Link for the main code](https://github.com/expert26111/MotherProcessBlockhain/blob/master/main.js)


![another way](https://github.com/expert26111/MotherProcessBlockhain/blob/master/postrequest.png)

The picture above shows how it works. Only the `main.js` is executed with the command `node main.js` and in the method `_addBlock`,  the mining will be done.

On other hand, if the project was implemented with Java, then we could have used Futures structure and the mining implement successfully. 






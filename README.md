# Blockchain
### <img src="https://upload.wikimedia.org/wikipedia/commons/a/a2/Simpleicons_Business_pencil.svg" width="18" height="18" /> Group LoanBorker 15: 

Yoana Dandarova

Manish Shrestha

-------------------------

### [Click here for video demonstration of the project.](https://drive.google.com/file/d/12dFBzNjHaZ5zsX1EUBjjVwzSxb-1nYfO/view?usp=sharing)

### Bash Script

To replicate it on a Linux machine via a script, [please download the interactive script from here.](https://raw.githubusercontent.com/shrestaz/Blockchain-SI/master/blockchain-interactive.sh)

- To make it executable, run the command: `chmod +x blockchain-interactive.sh`

- Then to execute the script, run: `./blockchain-interactive.sh`

-----------------------------------------------

#### *[Jump to learn how to interact with nodes. (View / add blocks and peers).](#Node)*
----------------------------------

# Table of Contents

1. [What is Blockchain?](https://github.com/shrestaz/Blockchain-SI#what-is-blockchain)

2. [Requirements](https://github.com/shrestaz/Blockchain-SI#requirements)

3. [Implementation](https://github.com/shrestaz/Blockchain-SI#implementation)

    3.1. [Architecture](https://github.com/shrestaz/Blockchain-SI#architecture) 
  
    3.2. [Block Structure](https://github.com/shrestaz/Blockchain-SI#block-structure) 
  
    3.3. [Block Hash](https://github.com/shrestaz/Blockchain-SI#block-hash) 
  
    3.4. [Storing of Blocks](https://github.com/shrestaz/Blockchain-SI#storing-the-blocks) 
  
    3.5. [Handling the new blocks](https://github.com/shrestaz/Blockchain-SI#handling-the-new-blocks) 
  
    3.6. [Nodes](https://github.com/shrestaz/Blockchain-SI#nodes)
  
4. [Alternative methods of mining](https://github.com/shrestaz/Blockchain-SI#alternative-methods-of-mining)

---------------------------------
 
 
## 1. What is Blockchain?

Blockchain is a technology that allows for fast, secure and transparent peer-to-peer transfer of digital goods including money and intellectual property. It allows for those information to be distributed and not copied. It can also be defined as an incorruptible digital ledger of transactions.

*While being originally formulated for digitally currency, “Bitcoin”, there has evolved other potential uses of this technology.*

Blockchain holds the information as a shared and continually reconciled database. Since it is shared and hosted by millions of computers simultaneously, it is truly public and easily verifiable. There doesn’t exist a centralized version which can be hacked and/or tampered with.

To summarize, blockchain consists of:

1. A network of nodes

2. Decentralization

3. Transparent and incorruptible

4. Distributed database

The use cases of this technology has spread like wildfire. It is used widely on online transactions. It is also implemented on day-to-day tools like GitHub and Google Docs, for example.

*Source: https://blockgeeks.com/guides/what-is-blockchain-technology/*

--------------------------------------------------

## 2. Requirements:

The following are the formal requirements for the project.

1. A peer-to-peer networking

2. A blockchain network consisting of 4 nodes.

3. An implementation with interactive API endpoint

4. Reproducible setup in a Linux based system with example nodes and test scenario

5. Demonstration of the running system

[A complete description provided for the requirements can be found here.](https://github.com/datsoftlyngby/soft2017fall-system-integration-teaching-material/blob/master/assignments/Blockchain.md)

----------------------------------------

## 3. Implementation:

In simple terms, our approach consists of a Node.js implementation with Express framework. It is instantiated by Docker-Compose as nodes with a peer-to-peer networking.
It also uses HTTP interface to interact with a node and Websockets to communicate with other nodes as a peer-to-peer connection.

### 3.1. Architecture

![Architecture](https://lh3.googleusercontent.com/A8pty_TVR6pp6EPdl9CFXRFJw37ieJ5uxn7VeuT-ZmTh_SXL-4JI3aIhO_R7F4gDbzNWKDP1n039wQ=s350 "Architecture")

The nodes exposes two webservers.

1. For user interaction : HTTP server

2. Peer-to-peer connection between nodes: Websocket server

### 3.2. Block Structure:

The structure of the block consists of:

- Index: An auto incrementing index as new blocks are added.
 
- Previous hash: Hash of the previous block to calculate new block’s hash. It ensure integrity when adding to the blockchain.

- Time stamp: the current timestamp from collected from the host machine to log the new block.

- Data: This data is the only manual input from the user. This can be any piece of information, the user wants to add to the blockchain.

- Hash: This is the hash value is calculated after all the above variables has been fulfilled, using SHA256 cryptographic hash function.

- Nonce: This is the Proof-of-Work part of the blockchain, calculated by the method `mineblock()`. This is calculated until the mined hash has a difficulty of level 4 (which can be adjusted as needed), meaning the first 4 values of the above mined hash equals to 0.

```sh
class Block {
    constructor(index, previousHash, timestamp, data, hash) {
        this.index = index;
        this.previousHash = previousHash;
        this.timestamp = timestamp;
        this.data = data;
        this.hash = this.calculateHash();
        this.mineBlock();
    }
}
```


![enter image description here](https://lh3.googleusercontent.com/gR1ffSlHGbPGq9JhC-2IGPiMqaPMNSzglGxzd93WbCNSffssOutCQ-oWdrg4AbTwPtiUV0lr-lCN0Q=s500 "block.png")


### 3.3. Block Hash:

The following method mines the hash of the block with difficulty 4. This is to keep the integrity of the data.

```sh
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

### 3.4. Storing the Blocks:
To store the blockchain, an in-memory Javascript array is used. The first block, also known as “Genesis block” is hardcoded.

```sh
var getGenesisBlock = () => {
    return new Block(0, "0", 1465154705, "I'm the first block. My friends call me Genesis block.", "0");
};

var blockchain = [getGenesisBlock()];

```

### 3.5. Handling the new blocks:
All the nodes are synchronized with each other. Meaning, each nodes listens for changes in other nodes. When a new block is mined and added to the chain in one of the nodes, the other nodes checks for its integrity. Only if the new block’s index number is greater than the existing block in the node’s chain, then the new block get’s added.

### 3.6. Nodes:
The user is able to interact with the nodes using an HTTP server.

```sh
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
To interact with the nodes, one can use programs such as POSTMAN or `curl` from the terminal. We'll use the latter for this example.

<a name="Node"></a>

The users can do the following with the nodes:

- List all blocks: using `/blocks` route

```curl http://localhost:3001/blocks```

- Create and add new block on a node with desired data: using `/mineblock` route

```sh
curl -H "Content-type:application/json" --data '{"data" : "I am the second block."}' http://localhost:3004/mineBlock
```
- List all peer: using `/peers` route

```curl http://localhost:3002/peers```

- Add new peer: using `/addPeer` route

```sh
curl -H "Content-type:application/json" --data '{"peer" : "ws://localhost:6001"}' http://localhost:3003/addPeer

```

*Source: https://medium.com/@lhartikk/a-blockchain-in-200-lines-of-code-963cc1cc0e54*

---------------------------------------

## 4. Alternative methods of mining

There has been a server created for returning a hash of difficulty 4. That means the mined hash will start with four 0s.

https://sleepy-cove-43230.herokuapp.com

![mother](https://github.com/expert26111/MotherProcessBlockhain/blob/master/herokurequest.png)
*Making a post request using PostMan* 

When making a post request, the block object is passed, mining process initiates and hash is returned.

**Alternate method 1:**

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

*Source: https://stackoverflow.com/questions/23667086/why-is-my-variable-unaltered-after-i-modify-it-inside-of-a-function-asynchron*<br>
*Source: https://stackoverflow.com/questions/37357843/run-node-script-from-within-a-node-script*<br>
*Source: https://stackoverflow.com/questions/13371113/how-can-i-execute-a-node-js-module-as-a-child-process-of-a-node-js-program*<br>
*Source: https://nodejs.org/api/process.html*<br>

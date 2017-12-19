# Blockchain assignment for System Integration

## Manish Shrestha

### To get started:

**You only need the Docker-compose.yml file**

The whole project is deployed on docker hub as an image.

1. Download the docker-compose.yml file.

2. On a terminal, run `docker-compose up`

> It will download the image from docker hub and instantiates the project.

3. Open 4 browser tabs. `localhost:3001`, `localhost:3002`, `localhost:3003` and `localhost:3004`

> It should have one block at this point, with hash starting with four 0's

4. To add blocks, run the following command.

`curl -H "Content-type:application/json" --data '{"data" : "I'm the second block. *dab*"}' http://localhost:3001/mineBlock`

> This will add a new block to the node **localhost:3001**. Change the data and node as needed.

5. Refresh all the nodes on the browser to see it have the same data.

6. Repeat step 4 with different data and on different nodes.
-----------------------
## Optional: Checking if p2p works

### Now we stop one of the nodes and see if the existing nodes still communicate with each other.

7. While running `docker-compose up`, open a new termainal and run `docker stop node 2`.

> This will stop node 2.

8. Repeat step 4 on one of the existing nodes, and check if your block syncs with other existing nodes.

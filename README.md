# Distributed Voting Application 
## Architecture

![](votingapp-architecture.png)

Following are the main components of this architecture:

<ul>
<li>A front-end web app in Python which lets you vote between two options and it writes the vote option into redis</li>
<li>A Redis cache which collects new votes</li>
<li>A .NET Core worker which consumes votes from redis cache and stores them in the postgres database</li>
<li>A Postgres database backed by a Docker volume</li>
<li>A Node.js webapp which shows the results of the voting in real time after reading these from the postgres database</li>
</ul>

_Note:_

The voting application only accepts one vote per client. It does not register votes if a vote has already been submitted from a client.

## Kubernetes Deployment

In order to deploy this stack to Kubernetes cluster, just run:

```
kubectl apply -f k8s-deployment/.
```

## Challenges

One of the common challenges with Kubernetes based development has been to develop these microservices locally without creating and pushing the docker images every time there is a change in the source code. This can easily be done using docker compose.

_Using Docker_ and defining your local development environment with _Docker Compose_ provides you with a number of benefits:
<ul>
<li>By running Redis and Postgres in a Docker container, you don’t have to install or maintain the software on your local machine</li>
<li>Your entire local development environment can be checked into source control, making it easier for other developers to collaborate on a project</li>
<li>You can spin up the entire local development environment with one command: docker-compose up</li>
</ul>

```
version: "3"

services:
  vote:
    build: ./vote
    command: python app.py
    volumes:
     - ./vote:/app
    ports:
      - "5000:80"
    networks:
      - front-tier
      - back-tier

  result:
    build: ./result
    #command: nodemon --inspect=0.0.0.0:5858
    command: nodemon server.js
    volumes:
      - ./result:/app
    ports:
      - "5001:80"
      - "5858:5858"
    networks:
      - front-tier
      - back-tier

```     
### Mount your code as a volume to avoid image rebuilds

Any time you make a change to your code, you need to rebuild your Docker image (which is a manual step and can be time consuming). To solve this issue, mount your code as a volume. Now rebuilds are no longer necessary when code is changed.

```
 vote:
    build: ./vote
    command: python app.py
    volumes:
     - ./vote:/app
```


_Happy coding !!!_

## Tasks

In the following tasks you will setup Docker images to run a server, a client and a logger application that you can find in the [`server`](./server), [`client`](./client) and [`logger`](./logger) directories.
These applications have the following functionalities:

- The client application selects random sentences from an input file and sends them to the server.
- The server processes the string in some way and sends it back to the client.
- The logger is notified whenever the server receives a message from a client, and keeps a counter of the number of events for each client.
  It prints its state regularly.

You may find it beneficial to create shell scripts or a Makefile to simplify building and running the applications and Docker networks/images in the tasks below, rather than typing the commands every time since they can become rather long.


### Task 1: Single Client and Server

First you will run the client and server in two separate containers, connected to the same virtual network.

1. Create a Docker network called `skynet` with the subnet `192.168.0.0/24` using the [`docker network` command](https://docs.docker.com/engine/reference/commandline/network/).
2. Create a `Dockerfile` and Docker image for the server application in the [`server`](./server) directory.
3. Create a `Dockerfile` and Docker image for the client application in the [`client`](./client) directory.
   The client Docker image must also contain the file [`loremipsum.txt`](./client/loremipsum.txt) to function correctly.

   *NOTE:* If `loremipsum.txt` is not in the same directory as the client binary inside the container, you need to specify its path with the `-src` command-line argument to the client application, e.g. as `-path=/path/to/loremipsum.txt`.

4. Run the server and the client with Docker and verify that they can communicate.
   Both containers should be in the `skynet` network and the server should use the IP address `192.168.0.10`.

### Task 2: Multiple Clients

In this task you will use the same setup as in Task 1, but will run several clients simultaneously.

1. Run the server image in the `skynet` network with the IP address `192.168.0.10` as in Task 1.
2. Run four clients in the `skynet` network and verify that they communicate with the server.
   Set the delay between the messages sent by the clients to 1000 ms.
   You can do this by passing the command-line argument `-delay=1000` when launching the client application.

   *HINT:* If your `Dockerfile` uses [`ENTRYPOINT` in the *exec form*](https://docs.docker.com/engine/reference/builder/#entrypoint) to launch the application, you can pass command-line arguments to the Docker image when using the `docker run` command.

### Task 3: Logger

In this task you will add the logger component from the [`logger`](./logger) directory to the network.
You can extend the setup from Task 1 or Task 2.

1. Create a `Dockerfile` and Docker image for the logger application in the [`logger`](./logger) directory.
2. Run the three components simultaneously in the `skynet` network.
   1. First run the logger (so that it is available when the server starts).
      The logger should have the IP address `192.168.0.20`.
   2. Run the server with the IP address `192.168.0.10`.
   3. Run one or more clients.
3. Confirm that the client(s) and server are communicating, and that the logger prints statistics regularly.

### Task 4: Docker Compose

When you need to run multiple Docker images together as in Task 3 it can quickly become difficult to manage.
In such cases it is convenient to automate the process.
The [Docker Compose](https://docs.docker.com/compose/) tool can be used for this purpose.
It allows you to specify things such as Docker networks, which images to build and run, which parameters to use, restarting behavior, etc. in a simple markdown format.

In this task you will use Docker Compose to manage a similar setup to the one from Task 3.
Create a file `docker-compose.yml` which defines the following:

1. Setup the Docker networks `skynet_frontend` with the subnet `192.168.1.0/24` and `skynet_backend` with the subnet `192.168.10.0/24`.
   We create two networks to simulate a situation where the clients are unable to communicate with the backend network of the service.
2. Start the logger.
   It should be part of the `skynet_backend` network with the IP address `192.168.10.20`.
3. Start the server.
   - The server should be part of both the `skynet_frontend` and the `skynet_backend` networks.
   - Its IP address in the `skynet_frontend` network should be `192.168.1.10`.
   - The server should contact the logger at `192.168.10.20:61112`, which can be done by passing the command-line argument `-logaddr=192.168.10.20:61112` when launching the server application.

   *HINT:* See the [`entrypoint`](https://docs.docker.com/compose/compose-file/#entrypoint) and [`command`](https://docs.docker.com/compose/compose-file/#command) configuration parameters.

4. Start four clients.
   - The delay should be set to 1000 ms using the `delay` command-line argument to the client application.
   - They should be part of the `skynet_frontend` network and communicate with the server at `192.168.1.10:61111`.
   This can be set by passing `-raddr=192.168.1.10:61111` as a command-line argument to the client application.
   - *NOTE:* To start several clients of the same definition with the `docker-compose` command, you can use the `--scale` command-line parameter.
   E.g. if you your client definition in `docker-compose.yml` is called `client`, you may use `docker-compose up --scale=4` to run 4 clients.
   Alternatively, you may copy-paste the configuration of a single client and change the name to "hard-code" several clients.
   You do not have to use the [`deploy`](https://docs.docker.com/compose/compose-file/#deploy) configuration or Docker Swarm unless you want to.

Afterwards, run the containers with `docker-compose` and verify that the clients communicate with the server and that the logger prints accurate statistics regularly.

- You should be able to start the containers with the command `docker-compose up` from the same directory as the `docker-compose.yml` file.
- *HINT:* You can run `docker-compose up --build` to rebuild the images every time.
  If not it will use images built from previous runs of `docker-compose` if they exist.

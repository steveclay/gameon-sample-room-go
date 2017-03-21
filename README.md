# Microservices with a Game On! Room

[![Codacy Badge](https://api.codacy.com/project/badge/Grade/e295bcca0c0342e3b7f32f316b62404a)](https://www.codacy.com/app/gameontext/gameon-room-go?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=gameontext/gameon-room-go&amp;utm_campaign=Badge_Grade)

[Game On!](https://gameontext.org/) is both a sample microservices application, and a throwback text adventure brought to you by the wasdev team at IBM. 
This application demonstrates how microservice architectures work from two points of view:

1. As a Player: Navigate through a network/maze of rooms, where each room is a unique implementation of a common API. Each room supports chat, and interaction with items (some of which may be in the room, some of which might be separately defined services as well).
2. As a Developer: Learn about microservice architectures and their supporting infrastructure by extending the game with your own services. Write additional rooms or items and see how they interact with the rest of the system.


## Introduction

This solution explores microservice architectures via a text adventure game. You can learn more about Game On! at [http://gameontext.org/](http://gameontext.org/).

This walkthrough will guide you through adding a room to a running Game On! microservices application.  You will be shown how to setup a container-based room that is implemented in the Go programming language.

### Installation prerequisites

Game On! when deployed using containers requires:

- [Bluemix account](https://console.ng.bluemix.net)
- [GitHub account](https://github.com/)
- [Docker](https://docs.docker.com/engine/installation/) - You will need Docker on your host machine to create a docker image to be pushed to IBM Bluemix Containers
- [IBM Containers CLI](https://console.ng.bluemix.net/docs/containers/container_cli_cfic_install.html) - This is required to manage IBM Containers from your local machine.  

## Create Bluemix accounts and log in
To build a Game On! room in Bluemix, you will first need a Bluemix account.

### Sign up and log into Bluemix and DevOps
Sign up for Bluemix at https://console.ng.bluemix.net and DevOps Services at https://hub.jazz.net. When you sign up, you'll create an IBM ID, create an alias, and register with Bluemix.

## Get Game On! ID and Shared Secret
For a new room to register with the Game-On application, you must first log into gameontext.org and sign in using one of several methods to get your **Game On! Id** and **Shared Secret**.

1.  Go to [https://gameontext.org/](https://gameontext.org/) and click **Enter**.
2.  Select an authentication method and log in with your user name and password for that type.
3.  View your user profile using the link in the top right.  It is either your username or a person icon.
4.  You should now see your **Game On! ID** and **Shared Secret** near the bottom of the page.

## Getting the source code

The source code is located in GitHub, navigate to our [source code](https://github.com/cfsworkload/gameon-room-go.git) and download the ZIP and unzip the code on your local machine.
Alternatively you can use the GitHub CLI to clone the repository with

`git clone https://github.com/cfsworkload/gameon-room-go.git`

## Configure your room
Our container will be expecting its parameters in the form of environment variables passed to it as part the docker run command. An example env file, example-container.env, has been provided and you must edit it to customize your settings. The file itself and container-startup.sh contain commentary regarding each of the environment variables, but in particular you will probably need to edit the following important settings:

- **CONTAINER_IP** This must be set to an available Bluemix public IP address.  You will be given instructions on getting this address in a following section.
- **GAMEON_ID** This is the **Game On! ID** provided by the microservices application after you login using your browser. (Click on the player icon in the upper right corner of the web page after logging in.)
- **GAMEON_SECRET** This is the **Shared Secret** provided by the microservices application after you login using your browser. (Click on the player icon in the upper right corner of the web page after logging in.)
- **ROOM_NAME** Set this to the name of your room.

## Make sure a public IP is available in your Bluemix space
This solution, when using Containers, requires a free public IP address. In order to determine if a public IP is available, you need to find the number of used and your max quota of IP addresses allowed for your space.

To find this information:

1. Log into Bluemix at https://console.ng.bluemix.net.
2. Select **DASHBOARD**.
3. Select the space you where you would like your Game On! room to run.
4. In the Containers tile, information about your IP addresses is listed.
5. Check that the **Public IPs Requested** field has at least one available IP address.

If you have an IP address available, you can request a new IP or use an existing available IP to start building your Game On! room. If all of your IP addresses have been used, you will need to unbind one. In either case, to manage your public IP addresses, install the CF IC plugin, which can be found at the website below.

[https://www.ng.bluemix.net/docs/containers/container_cli_ov.html#container_cli_cfic_installs](https://www.ng.bluemix.net/docs/containers/container_cli_ov.html#container_cli_cfic_installs)

Once installed:

1. Log into your Bluemix account and space.

  `cf ic login`  
2. List your current external IP addresses.

  `cf ic ip list`
3. If the list of external IP addresses contains an unused address, you can use that one for this solution.
4. If you are not at your limit of IP addresses and want a new one, request an IP address.

  `cf ic ip request`
5. If you need to make an IP address available, unbind an IP address currently in use.

  `cf ic ip unbind <public IP> <Container ID>`  

## Game On! room in a Container
To build a Game On! room in a container, first create the container locally in Docker using the provided Dockerfile, and then push the container onto your Bluemix container registry.

### Build the Docker container locally
1.  Start Docker to run Docker commands. e.g. with the Docker QuickStart Terminal.

2.  In the gameon-room-go folder, build the container using the docker build command:
    ```
    docker build -t registry.ng.bluemix.net/<your_namespace>/<imageName> .
    ```

    For example,

    ```
    docker build -t registry.ng.bluemix.net/mystuff/go-room:1.0 .
    ```

 Note the dot (.) at the end of the command. It is represents the current directory and should not be omitted.

3. To verify the image is now created, you can run `docker images`

### Push your container to your Bluemix container registry

1. Make sure you are logged in with the `cf ic login` command.

2. Push the container to Bluemix using the `docker push` command:
    ```
    docker push registry.ng.bluemix.net/<your_namespace>/<imageName>
    ```

   Use the same image name you used to create the local container. For example,

    ```
    docker push registry.ng.bluemix.net/mystuff/go-room:1.0
    ```

### Run the container on Bluemix
1. The container uses a startup script which is driven by environment variables. Edit example-container.env and change the environment variable to suit your needs as we will be using this to pass enviroment variable settings to the container.

 See the commentary in container-startup.sh and example-container.env for additional details.

2. Use the following command to run the container, using the same image name you used to create the local container.
    ```
    cf ic run --env-file=example-container.env -m 256 -d registry.ng.bluemix.net/<your_namespace>/<imageName>
    ```

   Or, to stay with the previous example,

    ```
    cf ic run --env-file=example-container.env -m 256 -d registry.ng.bluemix.net/mystuff/go-room:1.0
   ```

3. With the resulting container ID, use that value to bind to the IP you got earlier with the command:
    ```
    cf ic ip bind <IP> <Container ID>
    ```

## Access room on Game On!
Once the room is set up and it has registered with the microservices application, it will be accessible on [Game On!](https://gameontext.org/). It may take a moment for the room to appear.

1. Log in to [Game On!](https://gameontext.org/) using the authentication method you used to create your user ID and shared secret for the registered room.
2. Use the Game On! command `/listmyrooms` from The First Room, to see your list of rooms. Once your room is registered, it will appear in that list.
3. To get to your room, navigate through the network or go directly to it using the `/teleport` command from The First Room.

### List of host provided commands
The Game On! host provides a set a universal commands:
- **/exits** - List of all exits from the current room.
- **/help** - List all available commands for the current room.
- **/sos** - Go back to The First Room.

### The First Room commands
The First Room is usually where new users will start in Game On!. From there, additional commands are available and maintained by the Game On! microservices application. For the list of current commands use the `/help` command.

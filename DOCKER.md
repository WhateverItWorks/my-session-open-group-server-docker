# Deploying with Docker

Deploying with Docker allows you to run Session Group Server on any operating system without having to deal with OS-specific quirks.  
In some instances it might be more beginner-friendly, however it is also slightly more demanding in terms of resource usage.

## Requirements

- A server with [Docker](https://docker.com) installed
- git

## Setup

### Step 1: Clone the repository

```bash
git clone https://github.com/WhateverItWorks/my-session-open-group-server-docker.git session
cd session
```

### Step 2: Create or pull a docker image

```bash
sudo docker build -t session .
```

This will create an image for our container called `session`.   

### Step 3: Run the docker image

```bash
sudo docker run \
    -d \
    --name session \
    --restart always \
    -v ./session-open-group-data:/session-open-group-server/target/release/data \
    -p 9999:80 \
    massiveboxe/session-open-group
```

This will start an auto-restarting container named `session` on port 9999, with a persistent data volume called `session-open-group-data`, based on the image we've created in the step before.  
*If you've chosen to use a pre-built image*, replace  `session-open-group-server` with `massiveboxe/session-open-group`.

### Step 4: Configure

This command will connect you to the container's internal command line:

```bash
sudo docker exec -it session-open-group /bin/sh
```

And this one will take you to the path where the server binary is stored:

```bash
cd /session-open-group-server/target/release
```

You can now edit the configuration as you please. You can follow the [steps indicated in the README.md](https://github.com/WhateverItWorks/my-session-open-group-server-docker#CLI) and [the CLI reference](https://github.com/WhateverItWorks/my-session-open-group-server-docker#CLI), just **remember to use** `./session-open-group-server` **instead of** `session-open-group-server` **or** `/usr/bin/session-open-group-server`**.** Examples:

- Print the server URL: `./session-open-group-server --print-url`
- Become moderator of a room: `./session-open-group-server --add-moderator {your_session_id} {room_id}`

*Once you're done configuring*, exit the internal command line with this command:

```bash
exit
```

You don't need to restart the container after changing the configuration.

### Step 5: Enjoy

Your Session Open Groups server is ready.  
You can join it using the link sent on server startup or with the `--print-url` flag. However, **don't forget to add the port!** Since the server is running in a Docker container serving on port 9999, this is the correct URL:

```
http://[host_name_or_ip]:9999/[room_id]?public_key=[public_key]
```

(Replace 9999 with whatever port you chose in the `docker run` step if you didn't use 9999)


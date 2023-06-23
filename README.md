> Note: Session open group servers provide transit encryption, but open group messages are not encrypted while stored on the server; closed groups (which can be created within Session itself) are a better solution for high-security communications with groups of 100 or less people.

### my-session-open-group-server-docker-compose
Session open groups servers (SOGS) are group chat servers that can host thousands of chat participants.

### Security Audits:

- [Internet.nl](https://internet.nl/site/session.whateveritworks.org/2060148/)
- [HSTS Preload](https://hstspreload.org/)
- [SSL Labs](https://www.ssllabs.com/ssltest/analyze.html?d=session.whateveritworks.org)
- [Security Headers](https://securityheaders.com/?q=session.whateveritworks.org&hide=on&followRedirects=on)
- [pagespeed](https://pagespeed.web.dev/)
- [webbkoll](https://webbkoll.dataskydd.net/en)
- [ImmuniWeb](https://www.immuniweb.com/ssl/session.whateveritworks.org/uLlrAeMb/)
- [Hardenize](https://www.hardenize.com/report/session.whateveritworks.org/1686343966)
- [Mozilla.org](https://observatory.mozilla.org/)

### Manually (Recommended, If the Docker doesn't work for you!)
https://docs.oxen.io/oxen-docs/products-built-on-oxen/session/guides/open-group-setup

### Deploying with Docker

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



### CLI

With the server running in the background (e.g. using docker -d), you can run the following commands to perform various operations:

| Command                                 | Effect                            |
| --------------------------------------- |:---------------------------------:|
| `--add-room room_id room_name`          | to add a room                     |
| `--delete-room room_id`                 | to delete a room                  |
| `--add-moderator public_key room_id`    | to add a moderator to a room      |
| `--delete-moderator public_key room_id` | to delete a moderator from a room |
| `--print-url`                           | to print your server's URL        |

The open group server binary is normally located in `/usr/bin`, so to e.g. execute the `--print-url` command you'd run:

```
/usr/bin/session-open-group-server --print-url
```

### Documentation
All endpoints return the status code in the response body because that's the only way to propagate the status code back to the client when using onion requests.

## Endpoints

### GET /rooms/:room_id

| Header        | Required | Notes |
| ------------- | -------- | ----- |
| Authorization | No       |       |
| Room          | No       |       |

Returns information about the room with the given ID.

**Response:**

```
{
    status_code: u16,
    room: {
        id: String,
        name: String
    }
}
```

### GET /rooms

| Header        | Required | Notes |
| ------------- | -------- | ----- |
| Authorization | No       |       |
| Room          | No       |       |

Returns a list of all rooms on the server.

**Response:**

```
{
    status_code: u16,
    rooms: [
        {
            id: String,
            name: String
        },
        ...
    ]
}
```

### POST /files

| Header        | Required | Notes |
| ------------- | -------- | ----- |
| Authorization | Yes      | Basic |
| Room          | Yes      |       |

Store a file on the server.

**Expected body:**

```
{
    file: String // base64 encoded data
}
```

**Response:**

```
{
    status_code: u16
}
```

### POST /rooms/:room_id/image

| Header        | Required | Notes     |
| ------------- | -------- | --------- |
| Authorization | Yes      | Moderator |
| Room          | No       |           |

Set the image for a room.

**Expected body:**

```
{
    file: String // base64 encoded data
}
```

**Response:**

```
{
    status_code: u16,
    room_id: String
}
```

### GET /files/:file_id

| Header        | Required | Notes |
| ------------- | -------- | ----- |
| Authorization | Yes      | Basic |
| Room          | Yes      |       |

Get a file from the server.

**Response:**

```
{
    status_code: u16,
    result: String // base64 encoded data
}
```

### GET /rooms/:room_id/image

| Header        | Required | Notes |
| ------------- | -------- | ----- |
| Authorization | No       |       |
| Room          | No       |       |

Returns the preview image for the given group.

**Response:**

```
{
    status_code: u16,
    result: String // base64 encoded data
}
```

### GET /auth_token_challenge?public_key=string

| Header        | Required | Notes |
| ------------- | -------- | ----- |
| Authorization | No       |       |
| Room          | Yes      |       |

Get an auth token challenge. The requesting user generates a symmetric key from the ephemeral public key returned by the server and their private key, which can be used to decrypt the ciphertext and get the auth token.

**Response:**

```
{
    status_code: u16,
    challenge: {
        ciphertext: String, // base64 encoded data
        ephemeral_public_key: String // base64 encoded data
    }
}
```

### POST /claim_auth_token

| Header        | Required | Notes |
| ------------- | -------- | ----- |
| Authorization | Yes      | Basic |
| Room          | Yes      |       |

Claim the auth token in the `Authorization` header.

**Expected body:**

```
{
    public_key: String
}
```

**Response:**

```
{
    status_code: u16
}
```

### DELETE /auth_token

| Header        | Required | Notes |
| ------------- | -------- | ----- |
| Authorization | Yes      | Basic |
| Room          | Yes      |       |

Delete the auth token in the `Authorization` header.

**Response:**

```
{
    status_code: u16
}
```

### POST /compact_poll
| Header        | Required | Notes                                                                |
| ------------- | -------- | -------------------------------------------------------------------- |
| Authorization | No       | Authorization is handled on a room-by-room basis in the request body |
| Room          | No       |                                                                      |

Poll for new messages, new deletions and the current moderator list for multiple rooms all in one request.

**Expected body:**

```
{
    requests: [
        {
            room_id: String,
            auth_token: String,
            from_deletion_server_id: Option<i64>,
            from_message_server_id: Option<i64>
        },
        {
            ...
        }
    ]
}
```

**Response:**

```
{
    status_code: u16
    results: [
        {
            room_id: String,
            status_code: u16,
            deletions: [
                {
                    deletion_server_id: i64,
                    deleted_message_id: i64
                },
                {
                    ...
                }
            ]
            messages: [
                {
                    server_id: i64,
                    public_key: String,
                    timestamp: i64,
                    data: String,
                    signature: String
                },
                {
                    ...
                }
            ]
            moderators: [ "public_key_0", "public_key_1", "public_key_2", ... ]
        },
        {
            ...
        }
    ]
}
```

### POST /messages

| Header        | Required | Notes |
| ------------- | -------- | ----- |
| Authorization | Yes      | Basic |
| Room          | Yes      |       |

Store the given message on the server.

**Expected body:**

```
{
    public_key: Option<String>, // the public key of the sender
    timestamp: i64, // the sent timestamp of the message
    data: String, // the serialized protobuf
    signature: String // the base64 encoded message signature
}
```

**Response:**

```
{
    status_code: u16
    message: {
        server_id: String,
        public_key: Option<String>
        timestamp: i64
        data: String
        signature: String
    }
}
```

### GET /messages?from_server_id=i64&limit=u16

| Header        | Required | Notes |
| ------------- | -------- | ----- |
| Authorization | Yes      | Basic |
| Room          | Yes      |       |

Get messages from the server. If `from_server_id` is set only messages stored after that server ID are returned (limited to a maximum of 256 messages). Otherwise, if `limit` is set, the last `limit` messages stored on the server are returned (limited to a maximum of 256 messages).

**Response:**

```
{
    status_code: u16,
    messages: [
        {
            server_id: String,
            public_key: Option<String>, // the public key of the sender
            timestamp: i64, // the sent timestamp of the message
            data: String, // the serialized protobuf
            signature: String // the base64 encoded message signature
        },
        ...
    ]
}
```

### POST /delete_messages

| Header        | Required | Notes              |
| ------------- | -------- | ------------------ |
| Authorization | Yes      | Basic OR Moderator |
| Room          | Yes      |                    |

Deletes the messages with the given IDs from the server. The requesting user must either be the sender of the messages or have moderation permission.

**Expected body:**

```
{
    ids: [ 0, 1, 2, ... ], // the server IDs of the messages to delete
}
```

**Response:**

```
{
    status_code: u16
}
```

### DELETE /messages/:message_id

| Header        | Required | Notes              |
| ------------- | -------- | ------------------ |
| Authorization | Yes      | Basic OR Moderator |
| Room          | Yes      |                    |

Delete the message with the given ID from the server. The requesting user must either be the sender of the message or have moderation permission.

**Response:**

```
{
    status_code: u16
}
```

### GET /deleted_messages?from_server_id=i64&limit=u16

| Header        | Required | Notes |
| ------------- | -------- | ----- |
| Authorization | Yes      | Basic |
| Room          | Yes      |       |

Get deleted messages from the server. If `from_server_id` is set only deletions that happened after that server ID are returned (limited to a maximum of 256 deletions). Otherwise, if `limit` is set, the last `limit` deletions stored on the server are returned (limited to a maximum of 256 deletions).

**Response:**

```
{
    status_code: u16,
    ids: [
        {
            deletion_server_id: i64,
            deleted_message_id: i64
        },
        {
            ...
        }
    ]
}
```

### GET /moderators

| Header        | Required | Notes |
| ------------- | -------- | ----- |
| Authorization | Yes      | Basic |
| Room          | Yes      |       |

Get the full list of moderators.

**Response:**

```
{
    status_code: u16,
    moderators: [ "public_key_0", "public_key_1", "public_key_2", ... ]
}
```

### POST /block_list

| Header        | Required | Notes     |
| ------------- | -------- | --------- |
| Authorization | Yes      | Moderator |
| Room          | Yes      |           |

Ban the given public key from the server.

**Expected body:**

```
{
    public_key: String
}
```

**Response:**

```
{
    status_code: u16
}
```

### POST /ban_and_delete_all

| Header        | Required | Notes     |
| ------------- | -------- | --------- |
| Authorization | Yes      | Moderator |
| Room          | Yes      |           |

Ban the given public key from the server and delete all messages sent by them.

**Expected body:**

```
{
    public_key: String
}
```

**Response:**

```
{
    status_code: u16
}
```

### DELETE /block_list/:public_key

| Header        | Required | Notes     |
| ------------- | -------- | --------- |
| Authorization | Yes      | Moderator |
| Room          | Yes      |           |

Unban the given public key from the server.

**Response:**

```
{
    status_code: u16
}
```

### GET /block_list

| Header        | Required | Notes |
| ------------- | -------- | ----- |
| Authorization | Yes      | Basic |
| Room          | Yes      |       |

Get the full list of banned public_keys.

**Response:**

```
{
    status_code: u16,
    moderators: [ "public_key_0", "public_key_1", "public_key_2", ... ]
}
```

### GET /member_count

| Header        | Required | Notes |
| ------------- | -------- | ----- |
| Authorization | Yes      | Basic |
| Room          | Yes      |       |

Get the member count for the given room.

**Response:**

```
{
    status_code: u16,
    member_count: usize
}
```

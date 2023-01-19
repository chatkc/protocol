# The ChatKC Protocol

ChatKC is a protocol that originated from [MattKC](https://www.youtube.com/@MattKC)'s website [here](https://stream.mattkc.com/) and reverse engineered by [Breadpudding](https://github.com/cbpudding) and [janKonku](https://github.com/zegfarce) on January 16th, 2023. After playing around with the protocol for hours, MattKC found us hanging out and decided to tweak the protocol a bit to make things easier for us to work with. This document describes the protocol that was created as a result.

![Breadpudding: He's here but he's not active
MattKC: lol hey i didn't know people would still be here
Breadpudding: Oh heck
Platapai: yooo
MattKC: gonna work on some improvements
Breadpudding: Improvements? This webchat is already perfect.
Platapai: ^^^
janKonku: it's particularly nice for alternate clients
Breadpudding: That we may or may not be writing as we speak
janKonku: very little in the way of coding needed to connect to it and have it working
Breadpudding: Maybe reduce the rate limiting when you aren't streaming?](part1.png)

![MattKC: there is an argument for sending the time with the message, it could be nice even on here to be able to see exactly when a message was sent
MattKC: it's obviously recorded in the database but isn't sent to the client
Breadpudding: Wait, there's a database for this?
Breadpudding: Your poor server lol
janKonku: did you record the one test packet i sent
MattKC: yes the chat is backed by a database
janKonku: should be type 'test' :)](part2.png)

## Connectivity

In order to connect to a ChatKC server, you must establish a secure WebSocket connection to TCP port 2002 and send the "hello" event to the server. Once the server sends the initial burst of "message" events followed by "join" events, bidirectional communication can take place.

## Event Format

All events sent to and received from the server are JSON objects containing the `type` and `data` attributes. The `type` attribute will always be a string that describes the information found in the `data` attribute.

In addition to those two fields, events sent to the server are required to have the `auth` and `token` attributes. In the reference implementation of the server, `auth` is always set to the string "google" and `token` is set to your OAuth2 token.

## Event Types

### Clientbound Event Types

#### accepted

This event means that the server has acknowledged the last message you sent. The `data` for this type is an object with the `message` attribute set to the content of your last message.

Example:
```JSON
{
    "type": "accepted",
    "data": {
        "message": "Hello world!"
    }
}
```

#### authlevel

This event informs the client of their current authentication level. The `data` for this type is an object with the `value` attribute set to a number corresponding to your current authentication level.

Example:
```JSON
{
    "type": "authlevel",
    "data": {
        "value": 0
    }
}
```

#### chat

This event informs the client of a new message. The `data` for this type is an object with the following attributes:

| Name           | Type     | Description                                                                        |
| -------------- | -------- | ---------------------------------------------------------------------------------- |
| `auth`         | `number` | *Unknown*                                                                          |
| `author`       | `string` | The name of the user who created the message                                       |
| `author_color` | `string` | The hexadecimal color code of the user's preferred name color                      |
| `author_id`    | `number` | The identifier of the message author                                               |
| `author_level` | `number` | The authentication level of the message author                                     |
| `donate_value` | `string` | The donation amount given in USD (or an empty string for no donation)              |
| `id`           | `number` | The identifier of the message itself                                               |
| `message`      | `string` | The contents of the message                                                        |
| `reply`        | `number` | The identifier of the message being replied to (0 if none)                         |
| `time`         | `number` | The timestamp of when the message was created in milliseconds since the Unix epoch |

Example:
```JSON
{
    "type": "chat",
    "data": {
        "auth": 0,
        "author": "Breadpudding",
        "author_color": "B3DCF5",
        "author_id": 2268,
        "author_level": 0,
        "donate_value": "2.00",
        "id": 15225,
        "message": "Hello world!",
        "reply": 0,
        "time": 1674090083882
    }
}
```

#### delete

This event informs the client that the list of message identifiers given have been deleted from the server. The `data` for this type is an object with the `messages` attribute which is an array of message IDs.

Example:
```JSON
{
    "type": "delete",
    "data": {
        "messages": [
            15221,
            15222,
            15223,
            15224,
            15225
        ]
    }
}
```

#### getuserconf

This event informs the client of the user's current configuration. The `data` for this type is an object with the `color` and `name` attributes, where the `color` attribute is the user's name color in hexadecimal and the `name` attribute is the current username.

Example:
```JSON
{
    "type": "getuserconf",
    "data": {
        "color": "B3DCF5",
        "name": "Breadpudding"
    }
}
```

#### join

This event informs the client that a member has joined the chat. The `data` for this type is an object with the `name` attribute set to the member's name.

Example:
```JSON
{
    "type": "join",
    "data": {
        "name": "Breadpudding"
    }
}
```

#### part

This event informs the client that a member has left the chat. The `data` for this type is an object with the `name` attribute set to the member's name.

Example:
```JSON
{
    "type": "part",
    "data": {
        "name": "Breadpudding"
    }
}
```

#### servermsg

This event contains a message from the server to the client. This message is only visible to the client that received it and is typically used for error messages. The `data` for this type is an object with the `message` attribute.

Example:
```JSON
{
    "type": "servermsg",
    "data": {
        "message": "Available commands: commands, forum, forums, geoguessr, help, info, insta, instagram, time, timer, twitter, website, youtube, yt"
    }
}
```

#### status

This event informs the client of their current status. The `data` for this type is an object with the `status` attribute set to the current status.

Example:
```JSON
{
    "type": "status",
    "data": {
        "status": "authenticated"
    }
}
```

| Status Code       |
| ----------------- |
| `authenticated`   |
| `banned`          |
| `nameexists`      |
| `nameinvalid`     |
| `namelength`      |
| `nametimeout`     |
| `rename`          |
| `setuserconf`     |
| `unauthenticated` |

### Serverbound Event Types

#### getuserconf

This event is used to ask the server for the client's current configuration. There is no `data` field for this event.

Example:
```JSON
{
    "type": "getuserconf",
    "auth": "google",
    "token": "..."
}
```

#### hello

This event is used to introduce the client to the server. The `data` for this type is an object with the `last_message` attribute set to `-1`. Currently, we do not know why this data is sent.

Example:
```JSON
{
    "type": "hello",
    "data": {
        "last_message": -1
    },
    "auth": "google",
    "token": "..."
}
```

#### message

This event is used to send a message to the server. The `data` for this type is an object with the following attributes:

| Name    | Type     | Description                                                        |
| ------- | -------- | ------------------------------------------------------------------ |
| `reply` | `number` | The ID of the message that this message is a reply of (0 for none) |
| `text`  | `string` | The content of the message being sent                              |

Example:
```JSON
{
    "type": "message",
    "data": {
        "text": "Hello world!",
        "reply": 0
    },
    "auth": "google",
    "token": "..."
}
```

#### paypal

This event is used to donate money to the owner of the server.

Example:
```JSON
{
    "type": "paypal",
    "data": {
        // A lot of this isn't safe to put in documentation
    },
    "auth": "google",
    "token": "..."
}
```

#### setuserconf

This event is used to store the client's current configuration on the server. The `data` for this type is an object containing the `color` and `name` attributes.

Example:
```JSON
{
    "type": "setuserconf",
    "data": {
        "name": "Breadpudding",
        "color": "B3DCF5"
    },
    "auth": "google",
    "token": "..."
}
```

#### status

This event is used to request a status update from the server. The `data` for this type is an empty object.

Example:
```JSON
{
    "type": "status",
    "data": {},
    "auth": "google",
    "token": "..."
}
```

## Commands

Commands are actions performed by users using messages prefixed with an exclamation point.

### ban

Usage: `!ban <username>`

Summary: Bans a user from the chat

Minimum Authentication Level: *Unknown*

### help

Aliases: commands

Usage: `!help`

Summary: Lists all of the commands that the user has access to

Minimum Authentication Level: `0`

### info

Usage: `!info`

Summary: Displays information about the server

Minimum Authentication Level: `0`

### ipban

Usage: `!ipban <address>`

Summary: Blocks a specific IP address from sending messages

Minimum Authentication Level: *Unknown*

### mod

Usage: `!mod <username>`

Summary: Promotes a user to a moderator

Minimum Authentication Level: *Unknown*

### rm

Usage: `!rm <message id>`

Summary: Deletes a message from the chat

Minimum Authentication Level: *Unknown*

### time

Usage: `!time`

Summary: Displays the current time on the server

Minimum Authentication Level: `0`

### timer

Usage: `!timer <start/check/stop> <name>`

Summary: Creates, checks, or deletes a timer on the server

Minimum Authentication Level: `0`

### unban

Usage: `!unban <username>`

Summary: Reverts an existing ban on a user

Minimum Authentication Level: *Unknown*

### unmod

Usage: `!unmod <username>`

Summary: Demotes a user to an unprivileged member

Minimum Authentication Level: *Unknown*

## Message Formatting

*TODO*

### Emoji

*TODO*

### Mentions

*TODO*

### URLs

*TODO*

## Authentication Levels

The following is a list of the currently known authentication levels:

| Level | Meaning                              |
| ----- | ------------------------------------ |
| `0`   | An unprivileged member of the server |
| `100` | The owner of the server              |

### Permissions

| Level | `!ban` | `!ipban` | `!mod` | `!rm` | `!unban` | `!unmod` |
| ----- | ------ | -------- | ------ | ----- | -------- | -------- |
| `0`   | No     | No       | No     | No    | No       | No       |
| `100` | Yes    | Yes      | Yes    | Yes   | Yes      | Yes      |
# The ChatKC Protocol

ChatKC is a protocol that originated from [MattKC](https://www.youtube.com/@MattKC)'s website [here](https://stream.mattkc.com/) and reverse engineered by [Breadpudding](https://github.com/cbpudding) and [janKonku](https://github.com/zegfarce) on January 16th, 2023. After playing around with the protocol for hours, MattKC found us hanging out and decided to tweak the protocol a bit to make things easier for us to work with. This document describes the protocol that was created as a result.

## Connectivity

*TODO: WSS over port 2002*

## Message Format

*TODO: JSON message with a type string and data object*

## Message Types

### Inbound Message Types

#### accepted

*TODO*

#### authlevel

*TODO*

#### chat

*TODO*

#### delete

*TODO*

#### getuserconf

*TODO*

#### join

*TODO*

#### part

*TODO*

#### servermsg

*TODO*

#### status

*TODO*

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

### Outbound Message Types

#### getuserconf

*TODO*

#### hello

*TODO*

#### message

*TODO*

#### setuserconf

*TODO*

#### status

*TODO*

## Commands

Commands are special messages sent by users that are prefixed with an exclamation point. These are typically used for moderation purposes but can be used for other things later on.

### ban

*TODO*

### help

Aliases: commands

*TODO*

### info

*TODO*

### ipban

*TODO*

### mod

*TODO*

### rm

*TODO*

### time

*TODO*

### timer

*TODO*

### unban

*TODO*

### unmod

*TODO*

## Message Formatting

*TODO*

### Emoji

*TODO*

### Mentions

*TODO*

### URLs

*TODO*
# DynDNS

## Goal, Purpose, and Overview
I want to have my own solution for cases, when I want to have access to my personal projects/services in a decentralized way.

Real-world example: I plan to use my own gitlab instance with private unlimited repositories and projects.

Whenever I've tried to find solution, all of them are different from my vision, overcomplicated (_why I even need bind?_) or wrong.

The principle is simple:
  1. Whenever Docker is start on my machine, my containers are also start and connects to external server.
  2. Whenever connection is broken, it automatically recreated so for me or guest it looks like usual network issues.

**NOTE**: I want to use cheap server with just docker to make stable point across network.
          So, currently, example contains only docker compose configuration (even if I have
          my own kubernetes on Windows machine). But, anyway, this configuration is almost
          the same for other variants and, also, independent for connection itself (I mean
          we can use docker compose on server and kubernetes or docker swarm locally).

The architecture can be visualized as follows:
```
╔════════════════╗       ╔═════════════════╗
║ LOCAL COMPUTER ║ <===> ║ EXTERNAL SERVER ║
╚════════════════╝       ╚═════════════════╝
```

Quite simple, isn't?

Problems are occurring when we trying to determine exact principle of communication.

For tunnel I've diced to use SSH, because I don't need overcomplexity and don't want
to code product that I will not support in future.

```
╔════════════════════════╗                    ╔════════════════════════╗
║     LOCAL COMPUTER     ║ <=[ ssh tunnel ]=> ║    EXTERNAL SERVER     ║
╠════════════════════════╣                    ╠════════════════════════╣
║    ORCHESTRATOR        ║                    ║        ORCHESTRATOR    ║
║ ┌────────────────┐     ║                    ║     ┌────────────────┐ ║
║ │ TARGET SERVICE │     ║                    ║     │ REVERSE PROXY  │ ║
║ ├────────────────┤     ║                    ║     ├────────────────┤ ║
║ ┆ Shared Network ┆     ║                    ║     ┆ Shared Network ┆ ║
║ ├────────────────┤     ║                    ║     ├────────────────┤ ║
║ │ OPENSSH CLIENT │ ==> ║                    ║ ==> │ OPENSSH SERVER │ ║
║ └────────────────┘     ║                    ║     └────────────────┘ ║
╚════════════════════════╝                    ╚════════════════════════╝
```

So, to be precise, we interested in exact target-openssh and reverse-openssh configurations.
For reverse proxy I've diced to use nginx proxy manager, so I can avoid unstable solutions
like haproxy or traefik and have adequate configuration instead of label-like nonsence.

## Requirements

Basic requirements are:
1. Potato server that can handle docker.
2. Potato pc that can handle docker.

Since, we both meet requirements let talk about installation process.

## Installation

Clone this repository whatever you like (or download tar, I don't really care).
Generate public and private key for the future tunnel (never use your real key, be smart).

**NOTE**: Security setup is up to you, so you can adjust your needs to you special requirements.
          For example, we don't really need ssh keys at all, when we use all stuff in selfhosted,
          but we still may need to dynamically bind to some local mainframe.

Copy desired parts from repository (server to server, client to client respectfully).
For now, it's only docker-compose. Maybe, in some day (or if you are will to contribute),
I will make another example, but the core is same and simple.

Replace all parts that begins with `%REPLACE_WITH_*%`, then just start.

Enter to your DNS manager (no matter if it for local network or the usual one), redirect
all domains (like `*.myddns.mysite.com`) or specific (`app.mysite.com`) to the server.

Setup proxy pass to some port (in server orchestrator pod or those container network - in docker case).

Then use this port and your service port for ssh tunnel (don't worry, it will not be exposed if you're
not exposed your network).

## Basic working principle

OpenSSH (or its variants) client and server are "located" in same network, or, even better, in same pod,
so we can use same host (`127.0.0.1`) and cannot bind to wrong host or accidentally expose internals.

As for docker compose (which is simplest solution for selfhosted - at the first, and much more
mature than swarm - at the second), It also support sidecar analog. See `compose` directory for its config.
I believe, that configuration itself is not complicated, so you will understand all quickly.

## Recommendations

When first installing parts to server, make proxy pass to nginx proxy manager itself (`127.0.0.1:81`),
then remove port bind.

**NOTE**: 81 port is disabled by default, but you can make ssh tunnel as in client one and just setup
reverse proxy from your own pc.

## Instead of the end

If you setup all correctly, then at the next day, when you will turn on your pc (or exit sleep mode and etc),
your desired service will be online.

P.S. I haven't use any AI shit for writing this (or check this), but, probably, this documents is full of miss spelling. Feel free to contribute!

# Pylon Managed Gateway

This repository contains protobuf definitions and gRPC service headers for the various Pylon Gateway services.
The definitions can be used to generate API client code across many languages.

More information on the Pylon Gateway service follows.

### Navigation
- [Introduction](#introduction)
- [Features](#features)
- [Services](#services)
  - [Control API](#control-api)
  - [Event Stream (Dispatch)](#event-stream-dispatch)
  - [Cache API](#cache-api)
  - [REST Proxy](#rest-proxy)
- [Packages](#packages)
  - [Client Libraries](#client-libraries)
  - [Generated Code](#generated-code)
- [Service Map](#service-map)

## Introduction

The Pylon Managed Gateway is a one-stop-shop for interacting with the Discord Bot API and Gateway. By automatically scheduling bot shard connections across a cluster of nodes, the system maintains a highly-available event stream service, cache database, and REST proxy.

## Features

- Protobuf + gRPC APIs for robust, typed, and efficient communication.
- Fully typed Event, Cache, and REST gRPC services.
- Multiple bots with any number of shards may be scheduled on a cluster.
- Live shard migration: apply updates or change cluster topology without losing events!
- Efficient in-memory cache for Discord objects like members, channels, guilds, roles, messages, voice states, etc.
- API for fetching, listing, and querying cached Discord objects.
- Worker groups that clone, filter, and distribute events to dynamic groups of clients.
- Events include the previously cached state for an updated object, if applicable.
- Discord REST API Proxy that respects rate-limits and queues requests.
- Lock free, shared-nothing architecture.

## Services

![service header](https://cdn.discordapp.com/attachments/412407865911803906/803668445971218472/pylon-service-header.png)

### Control API

The Control API is used to administer the cluster. Common actions include adding bots, or configuring event stream worker groups.

### Event Stream (Dispatch)

Events from Discord, like `MESSAGE_CREATE` AND `GUILD_MEMBER_UPDATE`, are processed by the cache and distributed to Worker Groups. Clients connect to the Pylon Dispatch service via gRPC. A bi-directional stream is opened for client workers to receive and acknowledge events.

### Cache API

The Cache API is a very fast way to find data stored in a shard's cache. You can read available cache requests [here](https://github.com/pylonbot/pylon-gateway-protobuf/blob/main/gateway/v1/cache_service.proto). 

### REST Proxy

The Gateway's REST service provides a typed gRPC API over the Discord HTTP REST API. Requests are received by the Router service and directed to the Gateway node that owns the shard the HTTP request is for. Discord Rate-limits are transparently respected, and any requests that exceed the threshold are blocked and queued until ready to execute. You can read the available REST methods [here](https://github.com/pylonbot/pylon-gateway-protobuf/blob/main/gateway/v1/rest_service.proto). 

# Service Map + Example Bot Architecture

The diagram below features a broad overview of the internal services that make up the Pylon Gateway, including how a typical Discord bot or application may interact with Pylon Gateway's services.

![service map](https://cdn.discordapp.com/attachments/412407865911803906/803650350229225533/pylon-service-map.png)

# Packages

### Client Libraries
For writing applications that intend to receive Discord gateway events, you should use a fully-featured client library. These libraries intend provide the Pylon Managed Gateway connection logic and, in some cases, object-oriented wrappers around Discord models including methods for REST endpoints and cache requests.

- Java (https://github.com/pylonbot/pylon-gateway-client-java)
- More libraries coming soon!

### Generated Code
Pre-generated Protobuf and gRPC stubs are maintained for a handful of languages, based on user demand:

- Rust (https://github.com/pylonbot/pylon-gateway-protobuf-rust)
- JavaScript (coming soon!)
- Python (coming soon!)
- Golang (https://github.com/pylonbot/pylon-gateway-protobuf-go)
- Elixir (https://github.com/pylonbot/pylon-gateway-protobuf-elixir)

Use a language you don't see listed? Let us know! Alternatively, you may generate your own stubs using `protoc` with a plugin for your language.



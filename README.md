# Pylon Managed Gateway

This repository contains protobuf definitions and gRPC service headers for the various Pylon Gateway services.
The definitions can be used to generate API client code across many languages.

More information on the Pylon Gateway service follows.

### Navigation
- [Introduction](#introduction)
- [Features](#features)
- [Packages](#packages)
  - [Client Libraries](#client-libraries)
  - [Generated Code](#generated-code)
- [gRPC Services](#services)
  - [Control API](#control-api)
  - [Event Stream (Dispatch)](#event-stream-dispatch)
  - [Cache API](#cache-api)
  - [REST Proxy](#rest-proxy)
- [Gateway Internals](#gateway-internals)
  - [Gateway Service](#gateway-service)
  - [Operator Service](#operator-service)
  - [Router Service](#router-service)
  - [Dispatch Service](#dispatch-service)
    - [Worker Groups](#worker-groups)

## Introduction

The Pylon Managed Gateway is a one-stop-shop for interacting with the Discord Bot API and Gateway. By automatically scheduling bot shard connections across a cluster of nodes, the system maintains a highly-available event stream service, cache database, and REST proxy.

## Features

- Protobuf + gRPC APIs for robust, typed, and efficient communication.
- Fully typed Event, Cache, and REST gRPC services.
- Simple bot configuration, just add bot tokens and go!
- Multiple bots with any number of shards may be scheduled on a cluster.
- Control API for cluster administration
- Live shard migration: apply updates or change cluster topology without losing events!
- Efficient in-memory cache for Discord objects like members, channels, guilds, roles, messages, voice states, etc.
- API for fetching, listing, and querying cached Discord objects.
- Worker groups that clone, filter, and distribute events to dynamic groups of clients.
- Events include the previously cached state for an updated object, if applicable.
- Discord REST API Proxy that respects rate-limits and queue requests.
- Lock-free, shared-nothing architecture.

# Packages

### Client Libraries
For writing applications that intend to receive Discord gateway events, you should use a fully-featured client library. These libraries intend to provide the Pylon Managed Gateway connection logic and, in some cases, object-oriented wrappers around Discord models including methods for REST endpoints and cache requests.

- Java (https://github.com/pylonbot/pylon-gateway-client-java)
- More libraries coming soon!

### Generated Code
Pre-generated Protobuf and gRPC stubs are maintained for a handful of languages, based on user demand:

- Rust (https://github.com/pylonbot/pylon-gateway-protobuf-rust)
- JavaScript (coming soon!)
- Python (coming soon!)
- Golang (https://github.com/pylonbot/pylon-gateway-protobuf-go)
- Elixir (https://github.com/pylonbot/pylon-gateway-protobuf-elixir)

Use a language you don't see listed? Let us know! Alternatively, you may generate your stubs using `protoc` with a plugin for your language.

## Services

![pylon-service-header](https://github.com/user-attachments/assets/7bea23c6-b14f-4714-b506-c038deb75049)

### Control API

The Control API is used to administer the cluster. Common actions include adding bots or configuring event stream worker groups.

### Event Stream (Dispatch)

Events from Discord, like `MESSAGE_CREATE` AND `GUILD_MEMBER_UPDATE`, are processed by the cache and distributed to Worker Groups. Clients connect to the Pylon Dispatch service via gRPC. A bi-directional stream is opened for client workers to receive and acknowledge events. Read the section on [Worker Groups](#worker-groups) for more information.

### Cache API

Discord's strict rate-limits and lack of rich event data sometimes necessitate the need for local caching of data received from the real-time gateway.

The in-memory cache stores info on common Discord data models such as guilds, channels, roles, members, presences, voice-states, emoji, and messages. You can read available cache requests [here](https://github.com/pylonbot/pylon-gateway-protobuf/blob/main/gateway/v1/cache_service.proto). 

The cached state is maintained for the lifetime of a shard's session. When a shard's session is invalidated by Discord, the cache must be also invalidated and re-created with the next connection.

If your bot uses the Members or Presence intent, the Pylon Gateway can optionally request and cache all guild members from Discord as soon as possible.  Guild member chunks may be requested when a previously non-cached member is requested from the cache, or on-demand.

### REST Proxy

The Gateway's REST service provides a typed gRPC API over the Discord HTTP REST API. Requests are received by the Router service and directed to the Gateway node that owns the shard the request is destined for.

Discord Rate-limits are automatically respected, and any requests that exceed the threshold are kept waiting until ready to execute. When possible, permission checks and request validation is performed locally to reduce the number of invalid requests sent to Discord. You can read the available REST methods [here](https://github.com/pylonbot/pylon-gateway-protobuf/blob/main/gateway/v1/rest_service.proto). 

# Gateway Internals

The diagram below features a broad overview of the internal services that make up the Pylon Gateway, including how a typical Discord bot or application may interact with Pylon Gateway's forward-facing APIs.

![pylon-service-map](https://github.com/user-attachments/assets/b9c740b9-9222-424b-84de-f5b910ec0207)

## Gateway Service

The "Gateway" service is the core component of Pylon Gateway. Its main jobs are to maintain Shard tasks, forward Discord API requests, service Cache API requests, and buffer Discord events for client worker-groups. The Gateway service may run across a cluster of nodes, and it is the Operator Service's job to schedule and transfer shards across a cluster of healthy Gateway nodes.

![pylon-gateway-breakdown](https://github.com/user-attachments/assets/8828b360-8ee9-4e19-a126-7a25a4d02b48)

## Operator Service

The Operator Service is responsible for converging the cluster's state to the most optimal configuration. The service reads bot configurations and schedules shards across the Gateway Cluster. The Operator can detect unhealthy or failing nodes and reschedule or live-migrate shards to healthy Gateway nodes. Using live-migration, nodes may be safely decommissioned without losing events from Discord. As nodes are added or removed from the Gateway cluster, the operator re-balances shard assignments to ensure optimal performance.

## Router Service

Clients should not need to know about the shard topology of a Bot to make requests to the Discord API or fetch data from the Cache API. The Router service will route requests based on their Bot ID and Guild ID to the Gateway node that is running the shard that owns the requested resource. In the event of an in-progress shard transfer or transient outage, the Router service will transparently retry requests.

Some cache queries may need to fan out to multiple gateway nodes to serve one request. In this case, sub-requests are executed concurrently and merged at the Router before completing.

## Dispatch Service

To receive Discord events, clients must connect to the Dispatch service. Unlike traditional Discord connection sharding, workers may leave and join a worker group at any time. The Dispatch service maintains internal subscriptions to event streams (on the Gateway nodes) relevant to connected worker group clients. After authenticating, events are forwarded to worker group clients via a gRPC bidirectional stream. 

### Worker Groups

Worker groups are clusters of one or more workers that intend to receive events from the Pylon Gateway. They may be created via the Control API, and once configured emit event-streams that client worker clusters may connect to. Events are consistently distributed to workers based on Guild ID. Worker groups may subscribe to and optionally filter event streams by bot id, guild id, or event type. Requests to the Cache API and REST Proxy may too be optionally scoped to these same filters, preventing workers from making requests on behalf of bots or guilds they do not receive events for.

Multiple worker groups can be configured allowing for easy development of complex client applications. For example, a bot's premium version can run on a worker group separate from the main bot's features. Applications that offer "Whitelabel" services or custom bot accounts can simply add a bot to an existing worker group to bring it online.

# Pylon Managed Gateway

This repository contains protobuf definitions and gRPC service headers for the Pylon Managed Gateway service.
These definitions can be used to generate API client code across many languages.

More information on the Pylon Gateway service follows.

### Navigation
- [Introduction](#whatis)
- [Packages](#packages)
  - [Client Libraries](#client-libraries)
  - [Generated Code](#generated-code)
- [Service Map](#service-map)

# Introduction

Coming soon

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

# Service Map

![service map](https://cdn.discordapp.com/attachments/412407865911803906/803650350229225533/pylon-service-map.png)


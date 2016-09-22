# Console Server

- [Introduction](#introduction)
- [Working With Server](#working-with-server)
    - [Starting Server](#starting-server)
    - [Stopping Server](#stopping-server)
    - [Restart Behaviour](#restart-behaviour)

<a name="introduction"></a>
## Introduction

Kraken Server is the instance of console lister, that allows you to invoke all of your CLI commands remotely. It has to be created and run all the time alongside your application.

> {notice} Kraken Server is designed to work transparently, so your application won't be able to detect or react to its downtimes. On test or dev environments it is not an issue, however before publishing your application we strongy recommend you to read [deployment section](/docs/{{version}}/deployment).

<a name="working-with-server"></a>
## Working With Server

<a name="starting-server"></a>
### Starting Server

To start the server simply run:

    kraken.server

<a name="stopping-server"></a>
### Stopping Server

The only way to stop console server is to send SIGTERM signal to its process or close it via your system process manager. You can either store its pid on creation or search for process `kraken.server`.

<a name="restart-behaviour"></a>
### Restart Behaviour

It is totally safe to restart `kraken.server` instance during your application run, as it does not affect workflow of any of the containers.

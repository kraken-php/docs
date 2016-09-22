# Console Client

- [Introduction](#introduction)
- [Working With Commands](#working-with-commands)
    - [Invoking Commands](#invoking-commands)
    - [Listing Commands](#listing-commands)
    - [Getting Help About Command](#getting-help-about-command)
    - [Pinging Server](#pinging-server)
- [Basic Commands](#basic-commands)
    - [Command Types](#command-types)
    - [Command Families](#command-families)

<a name="introduction"></a>
## Introduction

Kraken Client is the command-line interface included with Kraken Framework. It ships with a number of helpful commands to manage and adjust your application on fly. It also allows you to add your own blocking or asynchronous commands.

<a name="working-with-commands"></a>
## Working With Commands

All of them commands provied with Kraken Client are either processed locally for commands that do not need to contact any of running runtime containers or remotely. Remote commands are in the first place send to Kraken Server, that then decides if it should deal with the command itself or send it corresponding runtime container. That's why it is important that you [start console server](/docs/{{version}}/console-server) before going further in this article.

<a name="invoking-commands"></a>
### Invoking Commands

You are able to invoke any of the provied commands using following syntax:

    php kraken [command] [options] [arguments]

Alternatively, if you have configured your php environment you can invoke it without using `php` binary, simply via:

    kraken [command] [options] [arguments]

<a name="listing-commands"></a>
### Listing commands

To list all commands available, use:

    kraken list

<a name="getting-help-about-command"></a>
### Getting Help About Command

To display help page regarding any command, use:

    kraken help [command]

<a name="pinging-server"></a>
### Pinging Server

To ping server and ensure connection exists and its properly configured, use:

    kraken server:ping

<a name="basic-commands"></a>
## Basic Commands

In basic configuration Kraken Console provides your application with several built-in commands that should be enough for fulfilling any runtime-based operation.

All commands have been created using following naming convention:

    [commandFamily]:[commandType]

To get detailed description about any command, please use `kraken help` command inside your application. In this article we will only talk about basic command families and types.

### Command Types

There are several command types that you can encounter while browsing Kraken Console help, but the most repeatable and important are `create`, `destroy`, `start`, `stop` and `status` types.

#### The `create` Type

    [commandFamily]:create

Create commands are used to create new instance of selected thing. Created thing is usually started after creation automatically so there's no need to call `start` -type command afterwards.

#### The `destroy` Type

    [commandFamily]:destroy

Destroy commands are used to destroy existing instance of selected thing. Referenced thing is usually stopped before destroying, so it could finish all remaining tasks.

#### The `start` Type

    [commandFamily]:start

Start commands are used to signal that some thing needs to be started. Command fires `start` event inside given thing, so it could start it processing - for example open its public ports.

> {notice} It is important to understand difference between `create` and `start` commands. The first one creates object from nothing while second uses existing instance to signal that it should start it work queue.

#### The `stop` Type

    [commandFamily]:stop

Stop commands are used to signal that some thing needs to be stopped. Command fires `stop` event inside given thing, so it could stop it processing - for example close its public ports. Stopping might be compared to maintenance mode.

> {notice} It is important to understand difference between `destroy` and `stop` commands. The first one destroys object completeley and frees it memory, while second signals only that existing instance should temporarily stop its processing, but keep all data.

#### The `status` Type

    [commandFamily]:status

Status commands are used - as the name suggests - for obtaining data about given thing state.

<a name="command-families"></a>
### Command Families

Command families represents the object or the part of the system on which given command should take place. Most important families that you should take look on in the first place are `project`, `arch`, `container`, `runtime`, `process`, `thread` and `server`.

#### The `project` Commands

Project commands refers to manipulating on whole project. It is important to know, that this kind of operations are performed by Kraken Server.

#### The `arch` Commands

Arch(itecture) commands are special type of commands that allows you to manipulating on selected container recursively. For example stoping container using `arch:stop` command will also stop all of its children.

#### The `container` Commands

Container commands are the commands that will be directly passed to specified container to execute. For example using `container:status A` command will send status request to container with alias `A`.

#### The `runtime` Commands

Runtime commands are the abstractions of `process` and `thread` commands that works on both process and thread based containers. The main difference between this family of command and the `container` commands are the fact, that `runtime` operate inidrectly using runtime ancestor. For example using `runtime:destroy A B` command will ask `A` container to destroy its child aliased as `B`.

#### The `process` Commands

Process commands behaves the same way as `runtime` commands, but works only for process-based containers.

#### The `thread` Commands

Thread commands behaves the same way as `runtime` commands, but works only for thread-based containers.

#### The `server` Commands

Server commands are the commands that works on Kraken Server. For example using `server:ping` will ping its instance.

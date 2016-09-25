# Network Component

- [Introduction](#introduction)
- [Features](#features)
- [Examples](#examples)
    - [Creating Network Server](#creating-network-server)
    - [Using Routes](#using-routes)
    - [Using Firewall](#using-firewall)
- [Important Classes & Interfaces](#important-classes-and-interfaces)
    - [NetworkServer](#network-server)
    - [NetworkServerInterface](#network-server-interface)
    - [NetworkConnection](#network-connection)
    - [NetworkConnectionInterface](#network-connection-interface)
    - [NetworkMessage](#network-message)
    - [NetworkMessageInterface](#network-message-interface)
    - [ServerComponentInterface](#server-component-interface)
- [Important Directories](#important-directories)
    - [Http](#http)
    - [Socket](#socket)
    - [Websocket](#websocket)

<a name="introduction"></a>
## Introduction

Network is a component that provides possibility to create standalone, asynchronous servers supporting various network protocols, including TCP, HTTP and WebSockets.

<a name="introduction"></a>
## Features

Network features:

<div class="dot-list" markdown="1">
- Asynchronous TCP server
- Asynchronous HTTP/1.1 server
- Asynchronous WebSocket server with support for RFC6455 and HyBi10 protocols
- Connections firewall
- HTTP request and response abstraction
- HTTP routing
- HTTP session provider
- Kraken Framework compatibility
</div>

Solver is an abstraction of a measure that have to be taken to solve problem which occurred.

<a name="examples"></a>
## Examples

This section contains examples and patterns that can be used with described component.

<a name="creating-network-server"></a>
### Creating Network Server

Network server can be used like this:

    $listener = new SocketListener('tcp://127.0.0.1:8080', $loop);
    $server   = new NetworkServer($listener);

After you are done with server it can be closed using:

    $server->stop();

<a name="using-routes"></a>
### Using Routes

When you have created instance of server, the routes can be added via `addRoute` method.

    $server->addRoute('/', new IndexAction); // HTTP GET /
    $server->addRoute('/other', new OtherAction); // HTTP GET /other
    $server->addRoute('/chat', new WsServer(null, new Chat)); // ws://localhost:8080/chat

<a name="using-firewall"></a>
### Using Firewall

Firewall is placed under `Kraken\Network\Socket\Component\Firewall`. If you are using default `NetworkServer` it has this component created by default. You can use it via `blockAddress` and `unblockAddress` methods, for example:

    $server->blockAddress('75.252.8.24'); // will deny access for this ip

<a name="important-classes-and-interfaces"></a>
## Important Classes & Interfaces

This section contains list of most important classes and interfaces shipped with this component. It **does not include all** classes and interface.

<a name="network-server"></a>
### NetworkServer

    class NetworkServer implements NetworkServerInterface

NetworkServer is an instance of universal server that allows route-based handling of various network protocols.

<a name="network-server-interface"></a>
### NetworkServerInterface

    interface NetworkServerInterface extends LoopResourceInterface

<a name="network-connection"></a>
### NetworkConnection

    class NetworkConnection implements NetworkConnectionInterface

NetworkConnection is an instance of incoming connection. Each connection represents one computer or another server connecting to your application.

<a name="network-connection-interface"></a>
### NetworkConnectionInterface

    interface NetworkConnectionInterface extends RatchetConnectionInterface

<a name="network-message"></a>
### NetworkMessage

    class NetworkMessage implements NetworkMessageInterface

NetworkMessage is an instance of incoming message or part of message in case of multi-part communication.

<a name="network-message-interface"></a>
### NetworkMessageInterface

    interface NetworkMessageInterface

<a name="server-component-interface"></a>
### ServerComponentInterface

    interface ServerComponentInterface

ServerComponentInterface is an interface that have to be implemented by each of server's components.

<a name="important-directories"></a>
## Important Directories

This section contains list of most important subdirectories placed inside this component. It **does not include all** its subdirectories.

<a name="http"></a>
### Http

Http directory contains HTTP server wrapper and all of its components.

<a name="socket"></a>
### Socket

Socket directory contains TCP server wrapper and all of its components.

<a name="websocket"></a>
### Websocket

Websocket directory contains Websocket server wrapper and all of its components.

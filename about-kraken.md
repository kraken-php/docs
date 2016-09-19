# About the Kraken

- [Description](#description)
- [Feature Highlights](#feature-highlights)
- [Modules](#modules)
- [Performance](#performance)
- [Licensing](#licensing)

<a name="description"></a>
## Description

Kraken is the first and only multi-processed, multi-threaded, fault-tolerant framework for PHP. It has been written to 
provide easy and reliable API for creating distributed applications using PHP. Kraken aims to solve typical problems of 
writing such applications and to provide developers with powerful yet elegant tools for dealing with them. 

The main focus of Kraken Framework is put on: 

<div class="dot-list" markdown="1">
- **Concurrency** : create systems that are asynchronous and concurrent by design,
- **Distribution** : divide your application into several containers and run them on multiple threads, processors or hosts,
- **Faul tolerance** : write systems that self-heal using remote and local supervision hierarchies,
- **Elasticity** : modify existing architecture in realtime without need to change in code,
- **High performance** : handle up to thousands of connections per second on each container,
- **Extensibility** : use available options to easily extend and adapt framework features for your needs.
</div>

Start writing applications that were previously marked as impossible or hard to implement in PHP right know. Servers, 
service-oriented architecture, agent-based models, games, complex daemons, socket programs, schedulers and much, much 
more - nothing is impossible with Kraken! 

<a name="feature-highlights"></a>
## Feature Highlights

Kraken features:

<div class="dot-list" markdown="1">
- Support for asynchronous programming using fully-featured event loop with multiple backgrounds.
- Support for event-driven architecture.
- Easy to understand and work with Promise-based API.
- Consistent multi-processing and multi-threading.
- Process and Thread abstraction as isolated message-driven containers.
- Built-in message routing system and IPC abstraction.
- Configurable local and remote supervision hierarchies.
- Centralized deployment and management.
- Extensible Console and Server interface.
- Asynchronous TCP and UDP sockets.
- Asynchronous Stream wrappers.
- Standalone HTTP and WebSocket server.
- Variety of IPC models.
- ReactPHP-compatibility adapters.
- ...and more.
</div>

<a name="modules"></a>
## Modules

Kraken Framework is fully modular and each of its components can be used separately. If for some reason you don't want to download full application stack, you can require any of the following components:

<div class="dot-list" markdown="1">
- **Kraken/Channel** provides IPC abstractions and routing message routing mechanisms,
- **Kraken/Config** provides Kraken/Filesystem -based configurator,
- **Kraken/Container** provides powerful dependency injection container, service providers and service container,
- **Kraken/Environment** provides Environement reader and configurator,
- **Kraken/Event** provides classes supporting and implementing event-driven architecture for PHP,
- **Kraken/Filesystem** provies Flysystem-based filesystem,
- **Kraken/Ipc** provides various IPC models including native sockets, ZMQ and more,
- **Kraken/Log** provies Monolog-based logging controller,
- **Kraken/Loop** provides fully featured event-loop with multiple backgrounds,
- **Kraken/Network** provides network protocol servers for TCP, HTTP, WebSocket and more,
- **Kraken/Promise** provides Promise/A+ implementation for PHP with forget-semantics for cancellations,
- **Kraken/Runtime** provides Thread and Process abstractions,
- **Kraken/Stream** provides asynchronous event-based stream wrappers,
- **Kraken/Supervision** provides supervision controllers and problem solvers,
- **Kraken/Throwable** provides custom Throwable hierarchy and extensive helper methods,
- **Kraken/Util** provides utility classes.
</div>

<a name="performance"></a>
## Performance

Kraken is able to emit millions of events and thousands of messages and connections per second using single container.
It is scalable for multiple processes and threads, faster than traditional PHP approach and able to handle same or 
higher amount of connections that Node.js.

<p align="center">
<img src="https://docs.google.com/uc?export=download&id=0B_FVuB10kPjVT21lY3JzVTRwT3c" width="882" height="334" />
</p>

<a name="licensing"></a>
## Licensing

After two years of intensive research Kraken has been created by Kamil Jamróz and is now developed further by its author and contributors as open source project. Kraken is licensed under the [MIT license](http://opensource.org/licenses/MIT).

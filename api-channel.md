# Channel Component

- [Introduction](#introduction)
- [Features](#features)
- [Concepts](#concepts)
- [Examples](#examples)
    - [Creating Channel](#creating-channel)
    - [Composing Channels](#composing-channels)
    - [Sending Messages](#sending-messages)
    - [Sending Requests](#sending-requests)
    - [Receiving Messages](#receiving-messages)
    - [Manual Receiving](#manual-receiving)
    - [Manual Pulling](#manual-pulling)
    - [Routing](#routing)
    - [Tweaking The Protocol](#tweaking-the-protocol)
- [Important Classes & Interfaces](#important-classes-and-interfaces)
- [Important Directories](#important-directories)
    - [Extra](#extra)
    - [Model](#model)
    - [Router](#router)

<a name="introduction"></a>
## Introduction

Channel is an event-based component that allows sending and receiving messages asynchronously. It provides abstraction for various IPC models and is designed to be used in multi-threaded, multi-processed systems. It provides complex routing mechanisms, protocols, message encoders and extends behaviour of decorated IPC models by implementing hearbeat mechanisms, reconnect mechanisms and allowing usage of both async and request-reply messaging patterns.

<a name="introduction"></a>
## Features

Channel features:

<div class="dot-list" markdown="1">
- Message-driven communication
- IPC models abstraction
- Support for sending asynchronous messages
- Support for request-reply pattern
- Built-in offline and online message buffers
- Built-in configurable protocol-based routing mechanisms
- Separation of input and output routers
- Heartbeat mechanism
- Reconnect mechanism
- Event-based API
- Promise-based helpers
- Kraken Framework compatibility
</div>

<a name="concepts"></a>
## Concepts

This section contains terminology, useful concepts and definitions, that might be helpful to learn to fully understand this component purpose.

### Channel

Channel is Kraken implementation of asynchronous mailbox. It handles incoming and outcoming messages send across threads and processes in asynchronous manner.

<a name="examples"></a>
## Examples

This section contains examples and patterns that can be used with described component.

<a name="creating-channel"></a>
### Creating Channel

To create a channel, you have to pass to the constructor various arguments. This example contains one of many possible configurations:

    $channel = new Channel(
        $id = 'test',
        new Model\Socket\Socket(
            $loop,
            [
                'id'       => $id,
                'endpoint' => 'tcp://127.0.0.1:3080',
                'type'     => Channel::BINDER,
                'host'     => $id
            ]
        ),
        new Router\RouterComposite([
            'input'  => new Router(),
            'output' => new Router()
        ]),
        new Encoder\Encoder(
            new \Kraken\Util\Parser\Json\JsonParser
        ),
        $loop
    );

> {tip} In this example `Kraken\Channel\Model\Socket\Socket` model has been used. It works on all PHP configurations and does not need any additional extensions. However, if you are able to use ZMQ extension on your server it is recommended to switch to `Kraken\Channel\Model\Zmq\ZmqDealer` as it is twice as fast, and uses less memory.

After creating a channel, it has empty routing, so all messages send or received will be automatically discarded. Before using Channel, remember to set it. The most basic configuration might be:

    $router = $channel->getRouter();
    
    // this will set a routing mechanism of incoming messages to accept all
    $input  = $router->getBus('input');
    $input->addDefault(function($params) use($channel) {
        $channel->pull(
            $params['alias'], 
            $params['protocol']
        );
    });
    
    // this will set a routing mechanism to try to send all messages directly
    $output = $router->getBus('output');
    $output->addDefault(function($params) use($channel) {
        $channel->push(
            $params['alias'], 
            $params['protocol'],
            $params['flags'],
            $params['success'],
            $params['failure'],
            $params['cancel'],
            $params['timeout']
        );
    });
    
Now channel is properly configured and can be run via `start` method and stopped with `stop` method.

    $channel->start();

> {tip} If you are using full technological stack of Kraken Framework, then the channels for communication are configured and created automatically, so you don't have to do this yourself.

<a name="composing-channels"></a>
### Composing Channels

Previously shown channel creation allows you to work with only one endpoint. To use multiple endpoints you have to create a channel instance for each one of them, and then compose them using `Kraken\Channel\ChannelComposite` class.

    $composite = new ChannelComposite([
        $id = 'name',
        [
            'bus1' => $channel1,
            'bus2' => $channel2,
        ],
        new RouterComposite([
            'input'  => new Router(),
            'output' => new Router()
        ]),
        $loop
    ]);

As you can see `ChannelComposite` has its own set of routers allowing you to create a routing mechanism that will decide which bus should be used for sending and receiving given message basing on this message context and protocol. To be able to use this composite channel, you have to create custom routing.

    $router = $channel->getRouter();
    
    // this will set a routing mechanism of incoming messages to accept all
    $input  = $router->getBus('input');
    $input->addDefault(function($params) use($channel) {
        $channel->pull(
            $params['alias'], 
            $params['protocol']
        );
    });
    
    // this will set a routing mechanism that will send all messages addresses with 'Bus1::' prefix 
    // to 'bus1' channel and 'Bus2::' prefix to 'bus2'. Other messages will be dropped.
    $output = $router->getBus('output');
    $output->addDefault(function($params) use($channel) {
        $alias = $params['alias'];
        
        if (strpos($alias, 'Bus1::') === 0) {
            $bus = $channel->getBus('bus1');
        } else if (strpos($alias, 'Bus2::' === 0) {
            $bus = $channel->getBus('bus2');
        } else {
            return;
        }
        
        $bus->send(
            $params['alias'], 
            $params['protocol'],
            $params['flags'],
            $params['success'],
            $params['failure'],
            $params['cancel'],
            $params['timeout']
        );
    });

Starting composite channel can also be done via `start` method. Remember that starting propagates to all of buses, so you don't have to start or stop all of them manually.

    $composite->start();

<a name="sending-messages"></a>
### Sending Messages

All messages passed via channels are asynchronous. To send a message use `send` or `push` method.

#### The `send` Method

The `send` method propagates message to output router, which then decides how message should be send. It is the method that should be used the most frequently.

    $status = $channel->send($addressee, $message);

#### The `push` Method

The `push` method sends message directly, without using router. Thanks to this behaviour it is more performant than `send` method, but is not available to resolve addressees which are not directly connected to the container.

    $status = $channel->push($addressee, $message);

<a name="sending-requests"></a>
### Sending Requests

While sending asynchronous messages is fast and lightweight thanks to not expecting any returned value, sometimes you might need to receive the answer. To get a returned value you should send request instead of simple message. Sending requests is possible via `send` and `push` methods, however it is more comfortable to use specially designed `Kraken\Channel\Extra\Request` object.

    $request = new Request($channel, $containerAddressee, $message); // request is being created
    $request
        ->call() // request is sent
        ->done(function($result) {
            printf("Request returned value: %s\n", $result);   
        });
 
<a name="receiving-messages"></a>
### Receiving Messages

When channel receives the message it emits corresponding `input` event.

    $channel->onInput(function($sender, $protocol) {
        printf("Received message from %s saying \"%s\".\n", $sender, $protocol->getMessage());
    });

> {notice} Sending and receiving requests do not fire `output` or `input` event.

<a name="manual-receiving"></a>
### Manual Receiving

To trigger manual receive of given message you can use `receive` method, which will then validate given message using `input router`.

    $channel->receive($sender, $protocol);

> {warning} This method uses routing mechanism to validate passed message, so it is not recommended to use it inside of routes, as it might create infinite loop.

<a name="manual-pulling"></a>
### Manual Pulling

To trigger manual pull of given message, use `pull` method. In contrast to `receive` it does not validate the message through routing mechanism, and accepts everything automatically.

    $channel->pull($sender, $protocol);

> {tip} This method is especially useful while creating input routing.

<a name="routing"></a>
### Routing

Routing can be done via resolving channel's router using `getRouter` method and then adding or removing routes.

    $router = $channel->getRouter();

#### The `addRule` Method

The `addRule` method of router enables you to create a routing rule. A rule contains a *matcher function* witch will be executed against each incoming message and *handler function* which will be executed if the matcher matches. Similarly to `catch` block, only the first match will be executed.

    $router->addRule(
        function($name, ProtocolInterface $protocol) {
            // returned value is the status of matcher execution
        },
        function($alias, ProtocolInterface $protocol, $flags, callable $success = null, callable $failure = null, callable $cancel = null, $timeout = 0.0) {
            // this will be executed if matcher returns true
        }
    );

#### The `addDefault` Method

The `addDefault` method of router allows you to create handler that will be executed if not one matcher returns true. In contrast to matcher, all of default methods are always executed.

    $router->addDefault(
        function($alias, ProtocolInterface $protocol, $flags, callable $success = null, callable $failure = null, callable $cancel = null, $timeout = 0.0) {
            // this will be executed if matcher returns true
        }
    );

#### Default Matchers & Handlers

The handler part of each rule is a complicated function. To make creating rules more user-friendly, a special `RouteHandler` class has been created that should be used instead of anonymous functions.

    $router->addDefault(
        new RuleHandler(function($params) {
            // this is much simplier, isn't it?
        });
    );

Kraken also provides you with a set of default matchers that can be found in `Kraken\Channel\Router\RuleMatch` namespace.

For example creating a rule that matches only messages sent from container with a name equal to `$name`, might be declared like this:

    $router->addRule(
        new RuleMatchName($name), // this matcher returns true if $protocol->getorigin() is equal to $name
        new RuleHandler(function($params) {
            // depending whether this rules applies to input or output channel, here we can either call 'pull' or 'push' method
        });
    );

<a name="tweaking-the-protocol"></a>
### Tweaking The Protocol

All of Kraken channel's method allows you to work on either string messages or `ProtocolInterface` ones. As described in [messaging article](/docs/{{version}}/messaging), channel used underneath only `ProtocolInterface` messages which he creates for you automatically from string messages. Sometime however, you might want to create this `ProtocolInterface` message manually.

Consider an example in which you want to create a message that will be passed from A container to C, via a B one. In default behaviour this cannot be done, because if you address the message to B, then C won't receive it, and if you address it to C, then the routing mechanism won't know what to do with this. To help you in situation like this you can create a `ProtocolInterface` manually and assign its fields. The assigned fields won't be overwritten by channel, but the fields that you will leave, still will be.

    $protocol = $channel->createProtocol($message);
    $protocol->setDestination('C'); // this will mark a message as addressed for C container
    $channel->send('B', $protocol); // this will send created message to B container, which he then will pass to C

There are more uses of tweaking protocol manually, but this article won't cover all of them, as it is quite advanced, mostly focused for internal development feature.

<a name="important-classes-and-interfaces"></a>
## Important Classes & Interfaces

This section contains list of most important classes and interfaces shipped with this component. It **does not include all** classes and interface.

<a name="important-directories"></a>
## Important Directories

This section contains list of most important directories existing inside of component. It **does not include all** directories.

<a name="extra"></a>
### Extra

Extra folder contains useful helpers, for example `Request` and `Response` classes.

<a name="model"></a>
### Model

Model folder contains all models that can be used with channels. Currently two models are supported `Socket` and `Zmq`.

<a name="router"></a>
### Router

Router folder contains all router itself, routes, rules and helpful matchers and handlers.

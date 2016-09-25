# Messaging

- [Introduction](#introduction)
- [Messaging Mechanism](#messaging-mechanism)
    - [Channel Definition](#channel-definition)
    - [Buses](#buses)
    - [Messaging Queue](#messaging-queue)
    - [Routing Mechanism](#routing-mechanism)
    - [Protocol Used](#protocol-used)
    - [Encoding](#encoding)
- [Working With Channels](#working-with-channels)
    - [Getting Channel Reference](#getting-channel-reference)
    - [Getting Buses](#getting-buses)
    - [Sending Asynchronous Messages](#sending-asynchronous-messages)
    - [Sending Requests](#sending-requests)
    - [Receiving Messages](#receiving-messages)
    - [Handling Connections](#handling-connections)

<a name="introduction"></a>
## Introduction

All Kraken messaging is done via specially-designed channels. They handle all incoming and outcoming messages automatically using custom-defined routing system. Each container has its own instance of channel and messaging queue. This article describes messaging mechanism and how it behaves.

<a name="messaging-mechanism"></a>
## Messaging Mechanism

<a name="channel-definition"></a>
### Channel Definition

Channel is Kraken implementation of asynchronous mailboxes. It deals with messages send across all Kraken architecture. Each channel usually consists of several [buses](#buses), [messaging queue](#messaging-queue), [routing mechanism](#routing-mechanism) and [encoders](#encoders).

<a name="buses"></a>
### Buses

Channel allows you to manage your application's multiple endpoints and various communication models using a single interface. To accomplish this, channel consists of multiple buses. Each bus represents single application endpoint. To manage passing messages via buses channel uses its own routing mechanism, that decides which buses should be used for any message using registered routing logic.

Each container has registered exactly one channel as `Channel` service. This channel composes two buses `master` and `slave`. The first one represents a connection to container's parent, while the second one is a connection to container's children.

<a name="messaging-queue"></a>
### Messaging Queue

Ordering of messages happens via enqueuing and dequeuing them using messaging queue built-in with channels. Enqueuing happens in the time-order of send operations, which means that messages sent from different containers may not have a defined order at runtime due to the apparent randomness of distributing containers across threads and processes. Sending multiple messages to the same target from the same container, on the other hand, will enqueue them in the same order. Receiving messages works in the same way.

<a name="routing-mechanism"></a>
### Routing Mechanism

Messages sent via channels route them to destination container using router and preferred routing mechanism. Different routing strategies can be used, according to your application's needs. Kraken comes with several useful routing strategies right out of the box. But, it is also possible to create your own.

<a name="protocol-used"></a>
### Protocol Used

Each message send and received is wrapped in `Kraken\Channel\Protocol\Protocol` structure which allows assigning context information for each message. This context is then used by routing mechanism to properly deal with message. Protocol is created and wrapped around each message automatically, however you are able to do this manually to tweak the context passed.

<a name="encoding"></a>
### Encoding

Channels send messages as strings. To be able to parse complicated structures each message is encoded before sending and decoded before receiving. The default encoder used in Kraken uses JSON format for parsing messages, but it can be changed to suit your application needs.

<a name="working-with-channels"></a>
## Working With Channels

<a name="getting-channel-reference"></a>
### Getting Channel Reference

To resolve channel reference `Channel` service can be used.

    $channel = $container->make('Channel');

> {tip} Sometimes you might want to use `master` and `slave` buses directly. You are able to resolve them via `Channel.Master` and `Channel.Slave` services. Sending and receiving messages directly using this services is more performant than `Channel`, but some routing mechanism might not be applied. Use this services only when you are 100% sure you know what you are doing.

<a name="getting-buses"></a>
### Getting Buses

To get `slave` or `master` bus from within channel, use `getBus` method.

    // getting slave
    $slave = $channel->getBus('slave');
    
    // getting master
    $master = $channel->getBus('master');

<a name="sending-asynchronous-messages"></a>
### Sending Asynchronous Messages

All messages passed via channels are asynchronous. To send a message use `send` or `push` method.

#### The `send` Method

The `send` method propagates message to output router, which then decides how message should be send. It is the method that should be used the most frequently.

    $status = $channel->send($containerAddressee, $message);

#### The `push` Method

The `push` method sends message directly, without using router. Thanks to this behaviour it is more performant than `send` method, but is not available to resolve addressees which are not directly connected to the container.

    $status = $channel->push($directAddressee, $message);

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

<a name="handling-connections"></a>
### Handling Connections

Whenever a channel detects connection or disconnection of other container, it emits either `connect` or `disconnect` event.

    $channel->onConnect(function($alias) {
        printf("Container %s has been connected.\n", $alias);
    });

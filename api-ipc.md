# Ipc Component

- [Description](#description)
- [Interface](#interface)
    - [Socket](#interface-socket)
    - [SocketListener](#interface-listener)
- [Examples](#examples)
    - [Using Socket and SocketListener Together](#using-socket-and-listener-together)

<a name="description"></a>
## Description

Kraken Ipc is a component that aims to provide asynchronous IPC models for your application.

Right now Kraken IPC supports:

<div class="dot-list" markdown="1">
- TCP sockets,
- UNIX sockets,
- ZMQ sockets and their subprotocols.
</div>

<a name="interface"></a>
## Interface

<a name="interface-socket"></a>
### Socket

    class Socket implements SocketInterface

    interface SocketInterface extends AsyncStreamInterface

#### SocketInterface::getLocalEndpoint()

    SocketInterface::getLocalEndpoint() : string

Get local endpoint matching following pattern [protocol://address:port].

#### SocketInterface::getRemoteEndpoint()

    SocketInterface::getRemoteEndpoint() : string

Get remote endpoint matching following pattern [protocol://host:port].

#### SocketInterface::getLocalAddress()

    SocketInterface::getLocalAddress() : string

Get local address matching following pattern [host:port].

#### SocketInterface::getRemoteAddress()

    SocketInterface::getRemoteAddress() : string

Get remote address matching following pattern [host:port].

#### SocketInterface::getLocalHost()

    SocketInterface::getLocalPort() : string

Get local host.

#### SocketInterface::getRemoteHost()

    SocketInterface::getRemotePort() : string

Get remote host.

#### SocketInterface::getLocalPort()

    SocketInterface::getLocalPort() : string

Get local port.

#### SocketInterface::getRemotePort()

    SocketInterface::getRemotePort() : string

Get remote port.

<a name="interface-listener"></a>
### SocketListener

    class SocketListener extends BaseEventEmitter implements SocketListenerInterface

    /**
     * @event connect : callable(object, SocketInterface)
     */
    interface SocketListenerInterface extends EventEmitterInterface, LoopResourceInterface, StreamBaseInterface

#### SocketListenerInterface::getLocalEndpoint()

    SocketListenerInterface::getLocalEndpoint() : string

Get local endpoint matching following pattern [protocol://address:port].

#### SocketListenerInterface::getLocalAddress()

    SocketListenerInterface::getLocalAddress() : string

Get local address matching following pattern [host:port].

#### SocketListenerInterface::getLocalHost()

    SocketListenerInterface::getLocalPort() : string

Get local host.

#### SocketListenerInterface::getLocalPort()

    SocketListenerInterface::getLocalPort() : string

Get local port.

<a name="examples"></a>
## Examples

<a name="using-socket-and-listener-together"></a>
### Using Socket and SocketListener Together

```
  $server = new SocketListener($endpoint, $loop);
  
  $server->on('connect', function(SocketListenerInterface $server, SocketInterface $conn) {
      $conn->on('data', function(SocketInterface $conn, $data) use($server) {
          $conn->write('secret answer!');
          $server->close();
      });
  });

  $client = new Socket($endpoint, $loop);
  $client->on('data', function(SocketInterface $conn, $data) {
      echo "Received answer=$data\n";
      $conn->close();
  });

  $client->write('secret question!');
```

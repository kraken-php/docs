# Stream Component

- [Introduction](#introduction)
- [Features](#features)
- [Concepts](#concepts)
- [Examples](#examples)
    - [Creating Asynchronous Streams](#creating-async-streams)
    - [Creating Streams](#creating-streams)
    - [Reading With Streams](#reading-with-streams)
    - [Writing With Streams](#writing-with-streams)
- [Important Classes & Interfaces](#important-classes-and-interfaces)
    - [AsyncStream](#async-stream)
    - [AsyncStreamInterface](#async-stream-interface)
    - [AsyncStreamReader](#async-stream-reader)
    - [AsyncStreamReaderInterface](#async-stream-reader-interface)    
    - [AsyncStreamWriter](#async-stream-writer)
    - [AsyncStreamWriterInterface](#async-stream-writer-interface) 
    - [Stream](#stream)
    - [StreamInterface](#stream-interface)
    - [StreamReader](#stream-reader)
    - [StreamReaderInterface](#stream-reader-interface)    
    - [StreamWriter](#stream-writer)
    - [StreamWriterInterface](#stream-writer-interface) 
    - [StreamSeeker](#stream-seeker)
    - [StreamSeekerInterface](#stream-seeker-interface) 

<a name="introduction"></a>
## Introduction

Stream is a component that provides consistent interface for managing and operating on stream including asynchronous writing and reading.

<a name="introduction"></a>
## Features

Stream features:

<div class="dot-list" markdown="1">
- Consistent interface for using streams
- Implementation of asynchronous and synchronous streams
- Kraken Framework compatibility
</div>

<a name="concepts"></a>
## Concepts

This section contains terminology, useful concepts and definitions, that might be helpful to learn to fully understand this component purpose.

### Asynchronous Processing

Asynchronous processing is an approach of processig instructions in non-blocking way, that only initiates the oprations without wait for response. The response is discovered later using specially designed mechanisms. You can learn more in [asynchronity section](/docs/{{version}}/asynchronity). 

<a name="examples"></a>
## Examples

This section contains examples and patterns that can be used with described component.

<a name="creating-async-streams"></a>
### Creating Async Streams

All of asynchronous streams' constructors are also the same. This applies to `AsyncStream`, `AsyncStreamReader` and `AsyncStreamWriter`.

    $stream = new AsyncStream($resource, $loop); // $loop have to be instance of Kraken\Loop\LoopInterface

<a name="creating-streams"></a>
### Creating Streams

All of synchronous streams' constructors are the same. This applies to `Stream`, `StreamReader`, `StreamWriter` and `StreamSeeker`.

    $stream = new Stream($resource);

<a name="reading-with-streams"></a>
### Reading With Streams

Regardless of chosen type of stream, might it be a synchronous or asynchronous one, the API is the same. The only difference is that in synchronous stream readers the `read` operation is blocking and returns read value while in asynchronous it is non-blocking, and the only way of obtaining read value is via event `data`.

    $reader->on('data', function($reader, $data) {
        // data event will be fired regardless of sync or async reading method
    });
    $reader->on('close', function() {
        // close event will also be fired
    });
    
    $data = $reader->read(); // this operation is blocking or non-blocking depending on stream type
    $reader->close();

<a name="writing-with-synchronous-streams"></a>
### Writing With Streams

Regardless of chosen type of stream, might it be a synchronous or asynchronous one, the API is the same.  The only difference is that in synchronous stream writers the `write` operation is blocking, while in asynchronous it is non-blocking.

    $writer->on('drain', function() {
        // drain event will be fired regardless of sync or async writing method
    });
    $writer->on('close', function() {
        // close event will also be fired
    });
    
    $writer->write($data); // this operation is blocking or non-blocking depending on stream type
    $writer->close();

<a name="important-classes-and-interfaces"></a>
## Important Classes & Interfaces

This section contains list of most important classes and interfaces shipped with this component. It **does not include all** classes and interface.

<a name="async-stream"></a>
### AsyncStream

    class AsyncStream extends Stream implements AsyncStreamInterface

AsyncStream is a concrete implementation of stream API allowing asynchronous reading and writing.

<a name="async-stream-interface"></a>
### AsyncStreamInterface

    interface AsyncStreamInterface extends AsyncStreamWriterInterface, AsyncStreamReaderInterface

<a name="async-stream-reader"></a>
### AsyncStreamReader

    class AsyncStreamReader extends StreamReader implements AsyncStreamReaderInterface

AsyncStreamReader is a concrete implementation of stream API allowing asynchronous reading.

<a name="async-stream-reader-interface"></a>
### AsyncStreamReaderInterface

    interface AsyncStreamReaderInterface extends StreamReaderInterface, LoopResourceInterface

<a name="async-stream-writer"></a>
### AsyncStreamWriter

    class AsyncStreamWriter extends StreamWriter implements AsyncStreamWriterInterface

AsyncStreamWriter is a concrete implementation of stream API allowing asynchronous writing.

<a name="async-stream-writer-interface"></a>
### AsyncStreamWriterInterface

    interface AsyncStreamWriterInterface extends StreamWriterInterface, LoopResourceInterface

<a name="stream"></a>
### Stream

    class Stream extends StreamSeeker implements StreamInterface

Stream is a concrete implementation of stream API allowing reading and writing synchronously.

<a name="stream-interface"></a>
### StreamInterface

    interface StreamInterface extends StreamWriterInterface, StreamReaderInterface

<a name="stream-reader"></a>
### StreamReader

    class StreamReader extends StreamSeeker implements StreamReaderInterface

StreamReader is a concrete implementation of stream API allowing reading synchronously.

<a name="stream-reader-interface"></a>
### StreamReaderInterface

    /**
     * @event data : callable(object, string)
     */
    interface StreamReaderInterface extends EventEmitterInterface, StreamSeekerInterface

<a name="stream-writer"></a>
### StreamWriter

    class StreamWriter extends StreamSeeker implements StreamWriterInterface

StreamWriter is a concrete implementation of stream API allowing writing synchronously.

<a name="stream-writer-interface"></a>
### StreamWriterInterface

    /**
     * @event drain : callable(object)
     */
    interface StreamWriterInterface extends EventEmitterInterface, StreamSeekerInterface

<a name="stream-seeker"></a>
### StreamSeeker

    class StreamSeeker extends BaseEventEmitter implements StreamSeekerInterface

StreamSeeker is a concrete implementation of stream API without possibility of reading or writing.

<a name="stream-seeker-interface"></a>
### StreamSeekerInterface

    /**
     * @event seek : callable(object, int)
     */
    interface StreamSeekerInterface extends EventEmitterInterface, StreamBaseInterface

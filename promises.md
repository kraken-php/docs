# Promises

- [Introduction](#introduction)
- [Implementing Promises](#implementing-promises)
    - [Description](#description)
    - [Promises & Events](#promises-and-events)
    - [States](#states)
    - [Changing States](#changing-states)
    - [Promise Forwarding](#promise-forwarding)
    - [Promise Nesting](#promise-nesting)
    - [Then vs Done](#then-vs-done)
- [Promise Helpers](#promise-helpers)
    - [The `all` Method](#the-all-method)
    - [The `any` Method](#the-any-method)
    - [The `map` Method](#the-map-method)
    - [The `race` Method](#the-race-method)
    - [The `reduce` Method](#the-reduce-method)
    - [The `some` Method](#the-some-method)

<a name="introduction"></a>
## Introduction

This article covers promises, deferreds and useful functions used for dealing with these kind of objects using Kraken Framework.

<a name="implementing-promises"></a>
## Implementing Promises

Kraken provides support for Promises implementing [Promise/A+](https://promisesaplus.com) specification extended with cancellation support using forget semantics and other useful promise-related concepts, such as joining, mapping, racing and reducing collections of promises.

<a name="description"></a>
### Description

Promises are a concurrency primitives that represent future value of an asynchronous operation. They simplify writting of asynchronous systems allowing sequential-like programming synatax. 

<a name="promises-and-events"></a>
### Promises & Events

While events provide a good interface for creating asynchronous applications, operating on them might become difficult and tiring in case of dynamically changing environment. It is easy to attach listeners once or follow a well-defined registering and unregistering scenario for given service. However, during creation of really dynamic and expand services, using events might become unintuitive and/or a chore.

Consider an example in which you have to create an asynchronous transaction containing of three also asynchronous services. You could try to do this with events with something like this:

    $listenerA = $serviceA->on('data', function($resource, $data) use($serviceA, $serviceB) {
        if ($serviceA->getResource() === $resource)
        {
            $serviceB->doSmth($data);
        }
    });
    
    $serviceB->once('doSmth', function($otherData) use($listenerA, $serviceC) {
        $listenerA->cancel();
        $serviceC->doSmthToo($otherData);
    });
    
    $serviceA->startOperation($initialData);

As you probably noticed, this easy sounding operation is extremely difficult to accomplish using events. Above example is probably even wrong and full of potential problems. Now, look how the same could be done using promises:

    Promise::doResolve($initialData)
      ->then(function($data) use($serviceA) {
          return $serviceA->startOperation($data);
      });
      ->then(function($data) use($serviceB) {
          return $serviceB->doSmth($data);
      })
      ->done(function($data) use($serviceC) {
          return $serviceC->doSmthToo($data);
      });

Its much simpler, isn't it? Promises are perfect for synchronizing execution of asynchronous operations.

<a name="states"></a>
### States

Promise may be in one of four states: pending, fulfilled, rejected or cancelled.

#### Pending Promise

Pending promise may transition to either the fulfilled, rejected or cancelled state.

#### Fulfilled Promise

Fulfilled promise may not transition to any other state and must have a value, which must not change.

#### Rejected Promise

Rejection promise may not transition to any other state and must have a reason, which must not change.

#### Cancelled Promise

Cancelled promise may not transition to any other state and must have a reason, which must not change.

<a name="changing-states"></a>
### Changing States

Only pending promise state can be changed. Changing its state can be treated as marking asynchronous operation as successful, unsuccessful or cancelled. These changes can be done via `resolve`, `reject` or `cancel` methods.

#### Resolving Promise

Resolving a Promise is a process of setting mixed value to **Pending Promise**, marking it as **Fulfilled** and firing its onFulfillment handler. Semantically it marks Promise as successful.

#### Rejecting Promise

Rejecting a Promise is a process of setting string value or Throwable to **Pending Promise**, marking it as **Rejected** and firing its onRejection handler. Semantically it marks Promise as failed with given reason for failure.

#### Cancelling Promise

Cancelling a Promise is a process of setting string value or Throwable to **Pending Promise**, marking it as **Cancelled** and firing its optional onCancellation handler. Semantically it marks Promise as cancelled meaning that the returned value does no longer matter and processing for which Promise awaits for should be terminated.

<a name="promise-forwarding"></a>
### Promise Forwarding

#### Resolution Forwarding

Resolvig a promise forwards resolution value from it to the next promise. Each call to `then` method returns a new promise that will resolve with the returned value of the previous one. This creates a promise chain.

    $deferred = new Deferred();
    $deferred
        ->getPromise()
        ->then(function($x) {
            // $x === 1
            // $x is the initial value passed to $deferred->resolve()
            return $x + 1;
        })
        ->then(function($x) {
            // $x === 2
            // this handler receives the return value of the previous promise
            return $x + 1;
        })
        ->done(function($x) {
            // $x === 3
            // this handler receives the return value of the previous promise and ends promise chain
            echo 'relved with ' . $x;
        });
    
    $deferred->resolve(1); // prints "resolved with 4"

#### Rejection Forwarding

Rejecting a promise behave in similar way and can be compared try-catch behaviour. When you catch an error or exception, you must rethrow it to propagate. If you don't do this promise the next promise will be fulfilled instead of being rejected.

    $deferred = new Deferred();
    $deferred
        ->getPromise()
        ->then(function($x) {
            throw new \Exception($x + 1); // throwing exception will reject the next promise
        })
        ->failure(function($x) {
            throw $x; // rethrowing propagates the rejection
        })
        ->failure(function(\Throwable $x) {
            return Promise::doReject(
                new \Exception($x->getMessage() + 1)
            );
        })
        ->done(null, function(\Throwable $x) {
            echo 'rejected with ' . $x->getMessage();
        });
    
    $deferred->resolve(1);  // Prints "rejected with 3"

#### Cancellation Forwarding

Cancelling promises propagates initial cancellation value and drops all returned values. This means, that cancelling a promise will mark the whole chain as cancelled. This method should be used carefully.

    $deferred = new Deferred();
    $deferred
        ->getPromise()
        ->abort(function($x) {
            // $x === 1
            // $x contains the initial value of cancellation
            return 2;
        })
        ->abort(function($x) {
            // $x === 1
            // $x contains the initial value of cancellation
            return 3;
        })
        ->done(null, null, function($x) {
            echo 'cancelled with ' . $x;
        });
    
    $deferred->cancel(1); // prints "cancelled with 1"

In comparison to resolving or rejecting a promise, cancellation also propagates upwards the chain, but cancels parent only when all of its children have been cancelled.

    $deferred = new Deferred();
    $parent = $deferred->getPromise();
    
    $child1 = $parent->then();
    $child2 = $parent->then();

    $child1->cancel(); // cancels only $child1
    $child2->cancel(); // cancels both $child2 and $parent

It is also important to know, that cancellation is propagated on promises created via any of static helper function provided with Kraken.

    $promise = Promise::all([
        $promise1,
        $promise2
    ]);
    
    $promise->cancel(); // it will cancel $promise, $promise1 and $promise2

<a name="promise-nesting"></a>
### Promise Nesting

One of the most interesting and useful things about promises, are the fact, that they allow deep nesting of other promises keeping all logic intact.

    $deferred = new Deferred();
    $deferred
        ->getPromise()
        ->then(function() {
            // let's assume that AsyncTask is an instance of promise
            return new AsyncTask();
        })
        ->then(function() {
            // this will be executed only after AsyncTask is resolved
        });

<a name="then-vs-done"></a>
### Then vs Done

At first `then` and `done` methods might seem very similar. However, there are important distinctions between the two. The intent of `then` is to transform a promise's value and to pass or return a new promise while the intent of `done` is to consume a promise's value, transferring all responsibility to your code. The `done` method shoul be called only in the outermost parts of promise chain.

    $deferred = new Deferred();
    $promise = $deferred->getPromise();
    
    $promise
        ->then(function($x) {
            // do something
        })
        ->done(function($x) {
            // completes promise chain so you cannot call then or done methods afterwards
        });
    
    $promise->resolve(1);

<a name="promise-helpers"></a>
## Promise Helpers

Kraken provides your application with the set of helpers to work on a collections of promises.

<a name="the-all-method"></a>
### The `all` Method

    Promise::all($promisesOrValues);

The `all` method returns a promise that will be resolved only once all the items in the passed collection have resolved. The resolution value of the returned promise will be an array containing the resolution values of each of the items in passed collection.

<a name="the-any-method"></a>
### The `any` Method

    Promise::any($promisesOrValues);

The `any` method returns a promise that will be resolved when any of the promises in passed collection have resolved. The resolution value of the returned promise will be the same as resolution value of a promse that have resolved.

<a name="the-map-method"></a>
### The `map` Method

    Promise::map($promisesOrValues, callable $mapFunc)

The `map` method is promise-based implementation of `array_map` function which can return either value or another promise.

<a name="the-some-method"></a>
### The `some` method

    Promise::some($promisesOrValues, $howMany);

The `some` method returns a promise that will be resolved when specified number of the supplied items in passed collection resolve. The resolution value of the returned promise will be an array containing the resolution values of the triggering items.

<a name="the-race-method"></a>
### The `race` method

    Promise::race($promisesOrValues);

The `race` method initiates a competitive race that allows one winner. Returns a promise which is resolved with the same value as the first settled promise.

<a name="the-reduce-method"></a>
### The `reduce` method

    Promise::reduce($promisesOrValues, callable $reduceFunc, $initialValue = null);

The `reduce` method is a promise-based implementation of `array_reduce` function which can return either value or another promise, the initial value can also be value or promise.

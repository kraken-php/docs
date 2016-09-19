# Component Promise

- [Description](#description)
- [Concepts](#concepts)
- [Interface](#interface)
    - [Deferred](#interface-deferred)
    - [Promise](#interface-promise)
    - [Functions](#interface-functions)
- [Examples](#examples)

<a name="description"></a>
## Description

Promises are a concurrency primitive with a proven track record and language integration in most modern programming languages.

Kraken provides support for Promises implementing [Promise/A+ specification](https://promisesaplus.com) extended with cancellation support using forget semantics and other useful promise-related concepts, such as joining, mapping, racing and reducing collections of promises.

<a name="concepts"></a>
## Concepts

### Deferred

Deferred represents a computation or unit of work that may not have completed yet. Typically, that computation will be something that executes asynchronously and completes computation at some point in the future.

### Promise

While deferred represents the computation itself, Promise represents the result of that computation. Each deferred object has a promise that acts as a placeholder for its actual result.

### Thenable

Thenable is an object or function that defines a then method.

### Promise States

Promise may be in one of four states: pending, fulfilled, rejected or cancelled.

*Pending Promise*
- may transition to either the fulfilled, rejected or cancelled state.

*Fulfilled Promise*
- may not transition to any other state,
- must have a value, which must not change.

*Rejected Promise*
- may not transition to any other state,
- must have a reason, which must not change.

*Cancelled Promise*
- may not transition to any other state,
- must have a reason, which must not change.

### Resolving Promise

Resolving a Promise is a process of setting mixed value to **Pending Promise**, marking it as **Fulfilled** and firing its onFulfillment handler. Semantically it marks Promise as successful.

### Rejecting Promise

Rejecting a Promise is a process of setting string value or Throwable to **Pending Promise**, marking it as **Rejected** and firing its onRejection handler. Semantically it marks Promise as failed with given reason for failure.

### Cancelling Promise

Cancelling a Promise is a process of setting string value or Throwable to **Pending Promise**, marking it as **Cancelled** and firing its optional onCancellation handler. Semantically it marks Promise as cancelled meaning that the returned value does no longer matter and processing for which Promise awaits for should be terminated.

<a name="interface"></a>
## Interface

<a name="interface-deferred"></a>
### Deferred

    class Deferred implements DeferredInterface

    interface DeferredInterface

#### Deferred::getPromise()

    DeferredInterface::getPromise() : PromiseInterface

Returns a Promise representing returned value of deferred operation.

#### Deferred::resolve()

    DeferredInterface::resolve(mixed $value = null) : PromiseInterface

Resolve pending promise.

#### Deferred::reject()

    DeferredInterface::reject(mixed $reason = null) : PromiseInterface

Reject pending promise.

#### Deferred::cancel()

    DeferredInterface::cancel(mixed $reason = null) : PromiseInterface

Cancel pending promise.

<a name="interface-promise"></a>
### Promise

    class Promise implements PromiseInterface

    class PromiseFulfilled implements PromiseInterface

    class PromiseRejected implements PromiseInterface

    class PromiseCancelled implements PromiseInterface

    interface PromiseInterface extends DeferredInterface

#### Promise::then()

    Promise::then(callable $onFulfilled = null, callable $onRejected = null, callable $onCancel = null) : PromiseInterface

Transform Promise's value by applying a function to the Promise's fulfillment, rejection or cancellation value. Returns a new promise for the transformed result.

#### Promise::done()

    Promise::done(callable $onFulfilled = null, callable $onRejected = null, callable $onCancel = null) : void

Consume the Promise's ultimate value if the promise fulfills or handle the ultimate error and cancellation. Returns null.

#### Promise::spread()

    Promise::spread(callable $onFulfilled = null, callable $onRejected = null, callable $onCancel = null) : PromiseInterface

Apply then transformation callbacks that automatically spreads received array into separate arguments. Returns a new promise for the transformed result.

#### Promise::success()

    Promise::success(callable $onSuccess) : PromiseInterface

Transform Promise's value by applying a function to the Promise's fulfillment value. Returns a new promise for the transformed result.

#### Promise::failure()

    Promise::failure(callable $onFailure) : PromiseInterface

Transform Promise's value by applying a function to the Promise's rejection value. Returns a new promise for the transformed result.

#### Promise::abort()

    Promise::abort(callable $onCancel) : PromiseInterface

Transform Promise's value by applying a function to the Promise's cancellation value. Returns a new promise for the transformed result.

#### Promise::always()

    Promise::always(callable $onSuccessOrFailure) : PromiseInterface

Apply cleanup handler that fires regardless of Promise resolution state and suppress return value.

#### Promise::isPending()

    Promise::isPending() : bool

Check if Promise is still pending.

#### Promise::isFulfilled()

    Promise::isFulfilled() : bool

Check if Promise is fulfilled.

#### Promise::isRejected()

    Promise::isRejected() : bool

Check if Promise is rejected.

#### Promise::isCancelled()

    Promise::isCancelled() : bool

Check if Promise is cancelled.

<a name="interface-functions"></a>
### Functions

Helper functions are implemented as static methods in PromiseStaticTrait that is used by each Promise state implementation.

All functions working on promise collections support cancellation. If you call cancel() on the returned promise, all promises in the collection are cancelled. If the collection itself is a promise which resolves to an array, this promise is also cancelled.

#### Promise::doResolve()

    Promise::doResolve(mixed $promiseOrValue = null) : PromiseInterface

Resolve Promise or value.

#### Promise::doReject()

    Promise::doReject(mixed $promiseOrValue = null) : PromiseInterface

Reject Promise or value.

#### Promise::doCancel()

    Promise::doCancel(mixed $promiseOrValue = null) : PromiseInterface

Cancel Promise or value.

#### Promise::all()

    Promise::all($promisesOrValues) : PromiseInterface

Return Promise that will resolve only once all the items in $promisesOrValues have resolved. The resolution value of the returned promise will be an array containing the resolution values of each of the items in $promisesOrValues.

#### Promise::race()

    Promise::race($promisesOrValues) : PromiseInterface

Initiate a competitive race that allows one winner. Returns a promise which is resolved in the same way the first settled promise resolves.

#### Promise::any()

    Promise::any($promisesOrValues) : PromiseInterface

Return a promise that will resolve when any one of the items in $promisesOrValues resolves. The resolution value of the returned promise will be the resolution value of the triggering item.

#### Promise::some()

    Promise::some($promisesOrValues, $howMany) : PromiseInterface

Return Promise that will resolve when $howMany of the supplied items in $promisesOrValues resolve. The resolution value of the returned promise will be an array of length $howMany containing the resolution values of the triggering items.

#### Promise::map()

    Promise::map($promisesOrValues, callable $mapFunc) : PromiseInterface

Map promises and/or values using specified $mapFunc.

#### Promise::reduce()

    Promise::reduce($promisesOrValues, callable $reduceFunc, mixed $initialValue = null)

Reduce Promises and/or values using $reduceFunc with $initialValue being Promise or primitive value.

<a name="examples"></a>
## Examples

### How to use Deferred

```
$deferred = new Deferred();

RunAsyncTask(function($result, $isOk) use($deferred) {
  if ($isOk) {
    $deferred->resolve($result);
  }
  else {
    $deferred->reject($result);
  }
});

return $deferred
  ->getPromise()
  ->then(
    function($value) {
      // success
    },
    function($reason) {
      // failure
    });
```


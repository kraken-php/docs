# Adding Runtime Listeners

- [Introduction](#introduction)
- [Adding Runtime Listeners](#adding-runtime-listeners)

<a name="introduction"></a>
## Introduction

<a name="adding-runtime-listeners"></a>
### Adding Runtime Listeners

To add runtime listeners from any place in your application, you should in first place resolve runtime object from service container and then bind event listener to it.

    $runtime = $container->make(RuntimeContainerInterface::class);
    
    $runtime->on('start', function() use($service) {
        $service->start();
    });
    $runtime->on('stop', function() use($service) {
        $service->stop();
    });

> {tip} The best place for registering runtime listeners service provider's `boot` method.

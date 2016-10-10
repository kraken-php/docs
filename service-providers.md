# Service Providers

- [Introduction](#introduction)
- [Using Service Providers](#using-service-providers)
    - [Registering & Unregistering Services](#registering-and-unregistering-services)
    - [Requiring & Providing Services](#requiring-and-providing-services)
    - [Booting Up](#booting-up)
    - [Exemplary Provider](#exemplary-provider)
- [Registering Providers](#registering-providers)
- [Registering Aliases](#registering-aliases)
- [Default Providers](#default-providers)
    - [Basic Providers](#basic-providers)
    - [Runtime Providers](#runtime-providers)
    - [Console Providers](#console-providers)

<a name="introduction"></a>
## Introduction

Service providers are the central place of Kraken application bootstrapping. All of Kraken's core and runtime services are bootstrapped using service providers. While creating your own application using Kraken Framework, it is recommended to split application into services and register them the same way.

The main purpose of using service providers includes registering service container bindings, schedulers, event listeners, timers, resources and other. In short, service providers are the place where configuring and combining your application should take place. You can read more about this process in [bootstrap section](/docs/{{version}}/bootstrap) or [dependency injection page](http://www.phptherightway.com/#dependency_injection).

This article focuses both on showing the way of creating your own, custom service providers and using existing, default ones.

<a name="using-service-providers"></a>
## Using Service Providers

All service providers extend the `Kraken\Container\ServiceProvider` class which contains a `register`, `unregister` and a `boot` methods.

<a name="registering-and-unregistering-services"></a>
### Registering & Unregistering Services

You should never attempt to register any event listeners, routes, or any other piece of functionality within the register method.

To register and unregister services from service container you should always use `register` and `unregister` methods. It is important to realize that long-running applications might want to dynamically change its architecture or reallocate memory, so unlike typical PHP providers, aside from `register` method Kraken offers also `unregister` one.

> {warning} While using `unregister` method might not be necessary for your application to run, avoiding its use might cause minor or major memory leak problems. The golden rule for using `unregister` method is simply to remember that everything previously registered should be unregistered there.

#### The Register Method

Within the `register` method, you should bind definitions into the service container. You should never register any event listeners, schedulers, timers or any other piece of functionality. Otherwise, you may accidentally refer to a service that is provided by a service provider, but has not been initialized yet.

#### The Unregister Method

Within the `unregister` method, you should contain reverse logic of `register` method, meaning that you should write code that unregisters all previously defined bindings using service container's `remove` method.

<a name="requiring-and-providing-services"></a>
### Requiring & Providing Services

As your project grows you might find yourself in situation that you have to register a service, that requires usage of another service, defined in separate provider. To help you with this kind of situations, Kraken allows for each provider to define its `requires` and `provides` protected fields.

The `requires` field should be an array containing the list of all services that your provider requires. This field will help Kraken to sort all registered providers in right order and ensure, that required services exists during registration of the provided that required them.

    protected $requires = [
        'App\Service\CustomService\CustomServiceInterface'  
    ];

The `provides` field should be an array containing the list of all services that your provider provides. Each class binding done by provider should be copied here, as this is the field that Kraken uses for sorting providers.

    protected $provides = [
        'App\Service\MyService\MyServiceInterface'  
    ];

<a name="booting-up"></a>
### Booting Up

Previously, it has been stated that registering and unregistering services should concern only binding of definitions. There are cases, that you additionally might need to apply some custom logic to your service, as - for example - creating event listeners that allows your service respond to Kraken runtime behaviour. This kind of logic should be put into `boot` method. The `boot` method is called after all other service providers have been registered, giving you access to all other services. 

<a name="exemplary-provider"></a>
### Exemplary Provider

To sum everything said in this article up to this point, here you can see exemplary service provider using all of provider features:

    class CustomProvider extends ServiceProvider implements ServiceProviderInterface
    {
        protected $requires = [
            LoopInterface::class
        ];
        
        protected $providers = [
            WampServerInterface::class
        ];
        
        protected function register(ContainerInterface $container)
        {
            $loop = $container->make(LoopInterface::class);
            $wamp = new WampServer($loop);
            
            $container->bind(WampServerInterface::class, $wamp);
        }
        
        protected function unregister(ContainerInterface $container)
        {
            $container->remove(WampServerInterface::class);
        }
        
        protected function boot(ContainerInterface $container)
        {
            // Since boot method is fired after everything has been registered, you dont need to put services used here in $requires field.
            $runtime = $container->make(RuntimeContainerInterface::class);
            $wamp    = $container->make(WampServerInterface::class);
            
            $runtime->onStart(function() use($wamp) {
                $wamp->start();
            });
            $runtime->onStop(function() use($wamp) {
                $wamp->stop();
            });
        }
    }

<a name="registering-providers"></a>
## Registering Providers

All service providers are registered in the `bootstrap.php` configuration file located in `data/bootstrap` directory. This file contains a providers array where you can list the class names or concrete objects of your service providers. By default, a set of Kraken core and runtime service providers are listed there. These providers bootstrap the basic Kraken components.

To register new provider add it at the end of the `providers` list:

    $providers = [
        // ...
        
        'App\Provider\MyCustomProvider'
    ];

<a name="registering-aliases"></a>
## Registering Aliases

Kraken allows you to create aliases for your services, so instead of calling them on container using their full qualified class name you would be able to call them using previously set alias. For example, in default framework configuration you can get channels by `Kraken\Channel\ChannelInterface` or by their alias `Channel`.

To register new alias add it at the end of the `aliases` list:

    $aliases = [
        // ...
        
        'myService' => 'App\Service\MyServiceInterface'
    ];

> {warning} The aliases cannot be used inside of service providers' `register` and `unregister` methods, as they are created after all providers have been registered.

<a name="default-providers"></a>
## Default Providers

Kraken Framework ships with set of default providers that can be used by your application. They are located inside in `Kraken\Root` namespace and registered in `bootstrap.php` file.

There three types of shipped providers:

<a name="basic-providers"></a>
### Basic Providers

Core providers contain bindings of default concrete object to interfaces and populate factories. They are stored inside `Kraken\Root\Provider` namespace.

<a name="runtime-providers"></a>
### Runtime Providers

Runtime providers contain configurations of services supplied by core providers. They are stored inside `Kraken\Root\Runtime\Provider` namespace.

<a name="console-providers"></a>
### Console Providers

Console providers are special providers used by Kraken console client and server. They are stored inside `Kraken\Root\Console\Provider` namespace.

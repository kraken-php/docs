# Container Component

- [Introduction](#introduction)
- [Features](#features)
- [Concepts](#concepts)
- [Examples](#examples)
    - [Binding Definition or Interface](#binding-definition-or-interface)
    - [Binding Factory Method](#binding-factory-method)
    - [Binding Default Params](#binding-default-params)
    - [Creating Aliases](#creating-aliases)
    - [Sharing Objects](#sharing-objects)
    - [Auto Wiring](#auto-wiring)
    - [Removing Definitions](#removing-definitions)
    - [Writing Service Provider](#writing-service-provider)
    - [Using Service Provider](#using-service-provider)
- [Important Classes & Interfaces](#important-classes-and-interfaces)
    - [Container](#container)
    - [ContainerInterface](#container-interface)
    - [ContainerAwareInterface](#container-aware-interface)
    - [ServiceProvider](#service-provider)
    - [ServiceProviderInterface](#service-provider-interface)
    - [ServiceRegister](#service-register)
    - [ServiceRegisterInterface](#service-register-interface)

<a name="introduction"></a>
## Introduction

Kraken container is both powerful dependency injection container and service container.

<a name="introduction"></a>
## Features

Container features:
<div class="dot-list" markdown="1">
- Support for binding objects, classes, params and factory methods to container
- Support for modifying container and removal of its definitions on fly
- Autoresolve for not defined classes or servies
- Autowiring for simple and nested dependencies
- Service providers with configurable requirements and providees
- Service registers
- Sorting algorithm to ensure right execution order of providers, based on their dependencies
- Kraken Framework compatibility
</div>

<a name="concepts"></a>
## Concepts

### Dependency Injection Container

Dependency injection container is an object that knows how to instantiate and configure objects used throughout application and all of their dependencies.

### Service Container

Service container is special case of dependency injection container that treats all of its bindings as abstract structures referred to as "services". Service container allows to create both concrete and virtual services. It is usually used together with set of service providers which it can manage, register, boot, sort and other things.

### Service Provider

Service provider is object which main purpose is to provide information how given service should be registered and booted into service container. 

### Service

Service is an abstract concept that usually referrs to an object which is created for a specific purpose and is used throughout your application whenever you need the specific functionality it provides.

<a name="examples"></a>
## Examples

<a name="biding-concrete-or-interface"></a>
### Binding Definition or Interface

    $container = new Container();
    $foo = new Foo();
    
    $container->bind(FooInterface::class, $foo);

<a name="biding-factory-method"></a>
### Binding Factory Method

    $container = new Container();
    
    $container->bind(FooInterface::class, function() {
        return new Foo();
    });

<a name="biding-default-params"></a>
### Binding Default Params

    $container = new Container();
    
    $container->wire(FooBar::class, [
        'constructor_option_1',
        'constructor_option_2'
    ]);

<a name="creating-aliases"></a>
### Creating Aliases

    $container = new Container();

    $container->alias(FooInterface::class, Foo::class);

<a name="sharing-objects"></a>
### Sharing Objects

    $container = new Container();
    
    // this will make Foo a singleton
    $container->share(Foo::class);

<a name="auto-wiring"></a>
### Auto Wiring

    $container = new Container();
    
    // this
    $bar = new Bar()
    $foo = new Foo($bar);
    
    // can be autoresolved by container using
    $foo = $container->make(Foo::class);

<a name="removing-definitions"></a>
### Removing Definitions

    $container = new Container();
    
    $container->bind(Foo::class, $foo = new Foo());
    $container->make(Foo::class); // === $foo
    
    $container->remove(Foo::class);
    $container->make(Foo::class); // !== $foo

<a name="writing-service-provider"></a>
### Writing Service Provider

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

<a name="using-service-provider"></a>
### Using Service Provider

    $container = new Container();
    $register  = new ServiceRegister($container);
    
    $register->registerProvider(new AProvider());
    $register->registerProvider(new BProvider());
    
    $register->boot();

<a name="important-classes-and-interfaces"></a>
## Important Classes & Interfaces

This section contains list of most important classes and interfaces shipped with this component. It **does not include all** classes and interface.

<a name="container"></a>
### Container

    class Container implements ContainerInterface

This class represents concrete implementation of dependency injection container.

<a name="container-interface"></a>
### ContainerInterface

    interface ContainerInterface extends ContainerReaderInterface, ContainerWriterInterface

<a name="container-aware-interface"></a>
### ContainerAwareInterface

    interface ContainerAwareInterface

Interface for container aware objects. Its main purpose is to provide setter and getter methods for setting and obtaining container instance withi objects composition.

<a name="service-provider"></a>
### ServiceProvider

    class ServiceProvider implements ServiceProviderInterface

Abstract service provider which should be extened by each provider used together with service container.

<a name="service-provider-interface"></a>
### ServiceProviderInterface

    interface ServiceProviderInterface

<a name="service-register"></a>
### ServiceRegister

    class ServiceRegister implements ServiceRegisterInterface

This class uses dependency injection container instance and behaves as service container

<a name="service-register-interface"></a>
### ServiceRegisterInterface

    interface ServiceRegisterInterface

# Registering Providers

- [Introduction](#introduction)
- [Registering Providers](#registering-providers)

<a name="introduction"></a>
## Introduction

<a name="registering-providers"></a>
### Registering Providers

To define custom providers for any runtime go to its corresponding bootstrap file and add them to `providers` list. It might look like this:

    $providers = $core->getDefaultProviders();
    $providers = array_merge($providers, [
        'App\Provider\CustomService\CustomServiceProvider',
        'App\Provider\OtherService\OtherServiceProvider',
    ]);

Remember that all of the custom providers need to extend `Kraken\Container\ServiceProvider` and implement `Kraken\Container\ServiceProviderInterface`.

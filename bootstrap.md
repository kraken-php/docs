# Bootstrap

- [Introduction](#introduction)
- [Directory Structure](#directory-structure)
- [Bootstrapping](#bootstrapping)
    - [Defining Providers](#defining-providers)
    - [Defining Services](#defining-services)
    
<a name="introduction"></a>
## Introduction

All of the bootstrap files for Kraken framework are stored in the `data/bootstrap` directory. Each file is documented, so feel free to look through them and get familiar with the default bootstrap configuration.

<a name="directory-structure"></a>
## Directory Structure

In contrast to many popular PHP frameworks, Kraken has been designed for multi-threaded and multi-processed applications. Thus, in comparison to them, Kraken allows you to create separate bootstrap file for each runtime.

There are several paths that will be checked for bootstrap file existence. Firstly, Kraken will check for `data/bootstrap/_unit_/_alias_/bootstrap.php` where '_unit_' can be one of `Process` or `Thread` depending on runtime unit type and '_alias_' is runtime's alias. If this file is not found, then default `data/bootstrap/_unit_/bootstrap.php` will be loaded. After finding first valid bootstrap file in previously depicted order, no other paths are checked.

> {tip} It is good idea to keep default bootstrap files for Processes and Threads universal, so you don't have to create separate for each unique runtime. Additional, non-default files should be created only when really needed.

<a name="bootstrapping"></a>
## Bootstrapping

Bootstrapping is the is the first process that runs when your application is started. It is responsible for loading the rest of the application piece by piece. In modern approach, these pieces are called services, and they are loaded, configured via [service providers](/docs/{{version}}/service-providers) and attached to [service container](/docs/{{version}}/service-container).

Kraken implements bootstrap as multi-stage process in which following operations are performed:

<div class="dot-list" markdown="1">
- Providers are sorted according to theirs' `requires` and `provides` fields
- Providers are being registered
- Provided services are being registered
- Providers are being booted
- Provided services are being booted
</div>

<a name="defining-providers"></a>
### Defining Providers

To define custom providers for any runtime go to corresponding bootstrap file and add them to 'providers' list. It might look like this:

```
$providers = $core->getDefaultProviders();
$providers = array_merge($providers, [
  'App\Provider\CustomService\CustomServiceProvider',
  'App\Provider\OtherService\OtherServiceProvider',
]);
```

<a name="defining-services"></a>
### Defining Services

Adding services can be accomplished in the same bootstrap file by adding them to 'aliases' list. It might look like this:

```
$aliases = $core->getDefaultAliases();
$aliases = array_merge($aliases, [
  'MyService'    => 'App\Service\My\MyServiceInterface',
  'OtherService' => 'App\Service\Other\OtherServiceInterface'
]);
```

As you probably noticed Kraken loads its default providers and services using `getDefaultProviders` and `getDefaultAliases` methods. You can read more about them in [providers page](/docs/{{version}}/service-providers).

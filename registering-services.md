# Registering Services

- [Introduction](#introduction)
- [Registering Services](#registering-services)

<a name="introduction"></a>
## Introduction

<a name="registering-services"></a>
### Registering Services

To add service aliases for any runtime go to corresponding bootstrap file and add them to `aliases` list. It might look like this:

    $aliases = $core->getDefaultAliases();
    $aliases = array_merge($aliases, [
        'MyService'    => 'App\Service\My\MyServiceInterface',
        'OtherService' => 'App\Service\Other\OtherServiceInterface'
    ]);

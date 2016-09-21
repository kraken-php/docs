# Config Component

- [Introduction](#introduction)
- [Features](#features)
- [Examples](#examples)
- [Important Classes & Interfaces](#important-classes-and-interfaces)
    - [Config](#config)
    - [ConfigInterface](#config-interface)
    - [ConfigFactory](#config-factory)
    - [ConfigFactoryInterface](#config-factory-interface)

<a name="introduction"></a>
## Introduction

Config is a component that allows to find, load, combine, autofill and validate configuration values of your application.

<a name="introduction"></a>
## Features

Config features:

<div class="dot-list" markdown="1">
- Support for accessing configuration values using dot-notation
- Support for isolating, merging and replacing multiple configurations
- Built-in overwrite handlers
- Kraken Framework compatibility
</div>

<a name="examples"></a>
## Examples

This section contains examples and patterns that can be used with described component.

<a name="reading-configuration-files">
### Reading Configuration Files

This example shows how configuration file can be read from file with a help from `Kraken\Filesystem` component.

    $path = __DIR__ . '/storage';
    $file = 'config.php';
    
    $adapterFactory = new FilesystemAdapterFactory();
    $configFactory  = new ConfigFactory(
        new Filesystem(
            $adapterFactory->create('Local', [ [ 'path' => $path ] ])
        ),
        [ '#' . $file . '#si' ]
    );
    
    $config = $configFactory->create();

<a name="reading-values">
### Reading Values

This example show how configuration values can be read from Config instance using dot notation.

    $value = $config->get('someOption.subOption.aKey', $defaultValue);

<a name="writing-values">
### Writing Values

    $config->set('someOption.subOption.aKey', 'anotherValue');

<a name="removing-values">
### Removing Values

    $config->remove($key = 'someOption.subOption.aKey');
    $value = $config->get($key); // returns null since this value is unset

<a name="merging-configuration-sets">
### Merging Configuration Sets

    $config->merge($newOptions = [
      'someOptions.option1' => 'value1',
      'someOptions.option2' => 'value2'
    ]);

<a name="using-overwrite-method">
### Using Overwrite Method

Using overwrite method allows you to define how given configuration set is going to be merged.

There are several options available:

<div class="dot-list" markdown="1">
- Isolation
- Merge
- Replace
- Reverse Isolation
- Reverse Merge
- Reverse Replace
</div>

This is how you can use them on a fly:

    $config->merge($newOptions, new Overwrite\OverwriteReplacer);

Alternatively Kraken allows you to set overwrite method as default, so you won't have to provide it on each `merge` call.

    $config->setOverwriteHandler(new Overwrite\OverwriteReplacer);

<a name="important-classes-and-interfaces"></a>
## Important Classes & Interfaces

This section contains list of most important classes and interfaces shipped with this component. It **does not include all** classes and interface.

<a name="config"></a>
### Config

    class Config implements ConfigInterface

Config is implementation of configuration-aware controller that allows writing and reading configuration using dot-notation.

<a name="config-interface"></a>
### ConfigInterface

    interface ConfigInterface extends ConfigReaderInterface, ConfigWriterInterface

<a name="config-factory"></a>
### ConfigFactory

    class ConfigFactory extends SimpleFactory implements ConfigFactoryInterface

ConfigFactory is a specially configured factory that allows creation of Config instances from `.php` configuration files.

<a name="config-factory-interface"></a>
### ConfigFactoryInterface

    interface ConfigFactoryInterface extends SimpleFactoryInterface


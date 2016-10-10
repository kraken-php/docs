# Configuration

- [Introduction](#introduction)
- [Directory Structure](#directory-structure)
    - [Importing Files](#importing-files)
- [Configuration](#configuration)
    - [Important Options](#important-options)
- [Configuration Variables](#configuration-variables)
    - [Default Variables](#default-variables)
    - [Inherited Variables](#inherited-variables)
    - [Environment Variables](#environment-variables)
    - [Functional Variables](#functional-variables)
    - [Defining Custom Variables](#defining-custom-variables)
- [Environment](#environment)

<a name="introduction"></a>
## Introduction

All of the configuration files for Kraken framework are stored in the `data/config` directory, while environment settings are placed in the `data/config.env` directory. Each option is documented, so feel free to look through the files and get familiar with the options available to you.

<a name="directory-structure"></a>
## Directory Structure

In contrast to many popular PHP frameworks, Kraken has been designed for multi-threaded and multi-processed applications. Thus, in comparison to them, Kraken allows you to create separate configurations and environment settings for each runtime.

There are several paths that will be checked for configuration files existence. Firstly, Kraken will check for `data/config/Process/$alias/config.php` or `data/config/Thread/$alias/config.php` depending on runtime unit type. If these files are not found, then `data/config/Process/config.php` or `data/config/Thread/config.php` will be fetched as default configuration files. After finding first valid configuration file in previously depicted order, no other paths are checked.

> {tip} It is good idea to keep default configuration files for Processes and Threads universal, so you don't have to create separate configurations for each unique runtime. Additional, non-default files should be created only when really needed.

<a name="importing-files"></a>
### Importing Files

As previously said, Kraken searches for first valid configuration file to load. However, it is possible to divide configuration options into smaller files and share them around. To be able to do this, you have to use `imports` option inside configuration file. It allows you to declare files that need to be loaded dynamically and merged to the configuration file that imported them. `imports` accepts array of `{ string resource, string mode }` elements in which resource should be valid, absolute path to another configuration file and mode one of `isolate`, `merge` or `replace` that specifies how merging should be done.

> {tip} Although resources needs absolute paths, you are able to use predefined variables as %basepath% or %datapath% that points to project root and project data directory.

Example

```
'imports' => [
  [ 'resource' => '%datapath%/config/CustomDirectory/config1.php', 'mode' => 'merge' ],
  [ 'resource' => '%datapath%/config/CustomDirectory/config2.php', 'mode' => 'replace' ]
]
```

<a name="configuration"></a>
## Configuration

All configuration options needs have either string or numerical value. Configuration file follows dot-notation access for multi-dimensional arrays.

**This:**

```
'option' => [
  'val1' => 'Value1',
  'val2' => 'Value2'
]
```

**Can be also written as:**

```
'option.val1' => 'Value1',
'option.val2' => 'Value2'
```

<a name="important-options"></a>
### Important Options

There are a few very important options that you have to take a look at. 

#### context

Context contains variables that will be passed to each of container children and be accessible for them as [inherited variable](#inherited-variables).

#### project

These options defines project settings. The most important options are `project.config.main.alias` and `project.config.main.name` as they define the root container to be executed on project creation. 

#### channel.channels

These options defines default IPC channels that container uses. By default Kraken defines `master` and `slave` channels for inter-container communication and `console` for console-root one.

> {tip} If you have ZMQ extension installed, you should consider changing the classes of each channels to use ZMQ instead of Socket model. ZMQ model is twice as fast.

<a name="configuration-variables"></a>
## Configuration Variables

Configuration variables are placeholders for values that will be replacced in configuration dynamically during initialization process. They allows you to create options that adapts automatically to runtime context.

Variables can be read using '%varName%' notation. Example:

```
'container.name' => 'container name is %name%'
```

<a name="default-variables"></a>
### Default Variables

Each runtime provides its configuration these default variables:

<div class="dot-list" markdown="1">
- **runtime** : can be one of 'Process' or 'Thread'
- **parent** : contains runtime parent's alias or null for root container
- **alias** : is equal to container alias
- **name** : is equal to container name
- **basepath** : points to project root directory
- **datapath** : points to project data directory
</div>

<a name="inherited-variables"></a>
### Inherited Variables

Inherited variables contains running container's parent's [context options](#important-options). They are accessible using 'inherited.' prefix, for example '%inherited.alias%'.

<a name="environment-variables"></a>
### Environment Variables

Environment variables put in 'config.env/.env' file can be accessed using 'env.' prefix, for example '%env.application_key%'.

<a name="functional-variables"></a>
### Functional Variables

Kraken allows your configuration to be parameterized using custom functions for retrieving values dynamically. Using '%func.myFunc%' will tell Kraken to execute current container's myFunc() method and replace variable with its returned value.

<a name="defining-custom-variables"></a>
### Defining Custom Variables

You can also define your own variables in 'vars' section.

```
'vars' => [
  'customVariable' => 'customValue'
]
```

And retrieve them by using `%customVariable%` syntax.

<a name="environment"></a>
## Environment

All of the environment settings are stored in the `data/config.env` directory. In current Kraken implementation this '.env' file is one and it is shared among all runtimes. 

Aside from configuring your application, it is very important to also tweak application [bootstrap files](/docs/{{version}}/bootstrap).

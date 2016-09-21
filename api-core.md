# Core Component

- [Introduction](#introduction)
- [Features](#features)
- [Examples](#examples)
    - [Getting Version](#getting-version)
    - [Getting Base Path](#getting-base-path)
    - [Getting Data Path](#getting-data-path)
- [Important Classes & Interfaces](#important-classes-and-interfaces)
    - [Core](#core)
    - [CoreInterface](#core-interface)
    - [CoreAwareInterface](#core-aware-interface)

<a name="introduction"></a>
## Introduction

Core fulfills the role of application kernel. It is composition of service container and register and is aware of its execution context.

<a name="introduction"></a>
## Features

Core features:

<div class="dot-list" markdown="1">
- Easy access to both service container and service register
- Context-aware interface
- Registering and booting mechanisms for application
- Kraken Framework compatibility
</div>

<a name="examples"></a>
## Examples

This section contains examples and patterns that can be used with described component.

<a name="getting-version">
### Getting Version

    $version = $core->getVersion();

<a name="getting-base-path">
### Getting Base Path

    $path = $core->getBasePath();

<a name="getting-data-path">
### Getting Data Path

    $path = $core->getDataPath();

<a name="important-classes-and-interfaces"></a>
## Important Classes & Interfaces

This section contains list of most important classes and interfaces shipped with this component. It **does not include all** classes and interface.

<a name="core"></a>
### Core

    class Core extends Container implements CoreInterface

Core is an implementation of application core.

<a name="core-interface"></a>
### CoreInterface

    interface CoreInterface extends ContainerInterface

<a name="core-aware-interface"></a>
### CoreAwareInterface

    interface CoreAwareInterface extends CoreSetterAwareInterface, CoreGetterAwareInterface

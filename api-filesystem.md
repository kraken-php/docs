# Filesystem Component

- [Introduction](#introduction)
- [Features](#features)
- [Concepts](#concepts)
- [Examples](#examples)
    - [Creating Filesystem](#creating-filesystem)
    - [Mounting Filesystems](#mounting-filesystems)
    - [Operating On Filesystem](#operating-on-filesystem)
    - [Listing Files](#listing-files)
    - [Listing Directories](#listing-directories)
- [Important Classes & Interfaces](#important-classes-and-interfaces)
    - [Filesystem](#filesystem)
    - [FilesystemInterface](#filesystem-interface)
    - [FilesystemManager](#filesystem-manager)
    - [FilesystemManagerInterface](#filesystem-manager-interface)
    - [FilesystemAdapterFactory](#filesystem-adapter-factory)
    - [FilesystemAdapterFactoryInterface](#filesystem-adapter-factory-interface)
- [Important Directories](#important-directories)
    - [Factory](#factory)

<a name="introduction"></a>
## Introduction

Filesystem is component that allows managing multiple filesystems and provides abstraction for various storage models.

<a name="introduction"></a>
## Features

Filesystem features:

<div class="dot-list" markdown="1">
- Abstraction for filesystem-related operations
- Mount manager for multiple filesystems
- Support for multiple storage models including clouds
- Kraken Framework compatibility
</div>

<a name="concepts"></a>
## Concepts

This section contains terminology, useful concepts and definitions, that might be helpful to learn to fully understand this component purpose.

### Filesystem

Filesystem is an abstraction of methods manipulating on local and remote file and directory structure.

<a name="examples"></a>
## Examples

This section contains examples and patterns that can be used with described component.

<a name="creating-filesystem"></a>
### Creating Filesystem

To create filesystem you have to pass storage model for it to operate upon.

    $fs = new Filesystem(
        (new FilesystemAdapterFactory)->create('Local');
    );

Or alternatively, since `Kraken 0.3.1`

    $fs = (new FilesystemFactory)->create('Local');

<a name="mounting-filesystems"></a>
### Mounting Filesystems

It is possible to mount multiple filesystems using `Kraken\Filesystem\FilesystemManager`. Mounted filesystems will then be accessible using proper prefix.

    $factory = new FilesystemFactory();
    $manager = new FilesystemManager([
        'local' => $factory->create('Local', [[ 'path' => __DIR__ ]]), // this fs will be available with local:// prefix
        'aws'   => $factory->create('AwsS3v3', [ $config ]); // this fs will be available with aws:// prefix
    ]);
    
<a name="operating-on-filesysem"></a>
### Operating On Filesystem

All operations on filesystem can be made via `Kraken\Filesystem\FilesystemInterface`.

#### Creating files:

    $fs->createFile($path, "some text\n", $fs::VISIBILITY_PRIVATE);

#### Writing:

Writing to files can be done by `write`, `prepend` and `append` methods, for example:

    $fs->append($path, "this will be added at the end\n");

#### Removing files:

    $fs->removeFile($path);

#### Creating directories:

    $fs->createDir($dir);

#### Removing directories:

    $fs->removeDir($dir);
    
For more options, please see `Kraken\Filesystem\FilesystemInterface`.

<a name="listing-files"></a>
### Listing Files

To list files use `getFiles` method.

    $files = $fs->getFiles($path, true, $filter = "#\.php$#si");

This example will list all `.php` files recursively in given path.

<a name="listing-directories"></a>
### Listing Directories

To list directories use `getDirectories` method.

    $dirs = $fs->getDirectories($path, $recursiveness, $filter);

<a name="important-classes-and-interfaces"></a>
## Important Classes & Interfaces

This section contains list of most important classes and interfaces shipped with this component. It **does not include all** classes and interface.

<a name="filesystem"></a>
### Filesystem

    class Filesystem implements FilesystemInterface

Loop is an abstraction layer for managing filesystem structure using various storage models. It wraps around the PHP League filesystem providing new, extended and Kraken-compatible interface, which has been redone with consistency in mind.

<a name="filesystem-interface"></a>
### FilesystemInterface

    interface FilesystemInterface

<a name="filesystem-manager"></a>
### FilesystemManager

    class FilesystemManager implements FilesystemManagerInterface

FilesystemManager is a special concrete object allowing you to create composition of filesystems and referencing all of them using one object.

<a name="filesystem-manager-interface"></a>
### FilesystemManagerInterface

    interface FilesystemManagerInterface extends FilesystemInterface

<a name="filesystem-adapter-factory"></a>
### FilesystemAdapterFactory

    class FilesystemAdapterFactory extends Factory implements FilesystemAdapterFactoryInterface

This factory allows you to create any filesystem adapter wrapping corresponding PHP League model.

<a name="filesystem-adapter-factory-interface"></a>
### FilesystemAdapterFactoryInterface

    interface FilesystemAdapterFactoryInterface extends FactoryInterface
    
<a name="important-directories"></a>
## Important Directories

This section contains list of most important directories existing inside of component. It **does not include all** directories.

<a name="factory"></a>
### Factory

Factory folder contains set of factories for creating variety of League Filesystem adapters for using Filesystem with cloud storage. All of them might be created using `Kraken\Filesystem\FilesystemAdapterFactory`.

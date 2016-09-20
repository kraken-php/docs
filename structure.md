# Project Structure

- [Introduction](#introduction)
- [App Directory](#app-directory)
    - [`Process` Directory](#process-directory)
    - [`Thread` Directory](#thread-directory)
- [Data Directory](#data-directory)
    - [`autorun` Directory](#autorun-directory)
    - [`bootstrap` Directory](#bootstrap-directory)
    - [`config` Directory](#config-directory)
    - [`config.env` Directory](#config-env-directory)
    - [`log` Directory](#log-directory)
    - [`storage` Directory](#storage-directory)
- [The kraken File](#kraken-file)
- [The kraken.server File](#kraken-server-file)

<a name="introduction"></a>
## Introduction

The default Kraken Framework structure is intended to provide a great starting point for any application. Kraken imposes almost no restrictions on where classes are located as long as they can be loaded via Composer.

<a name="app-directory"></a>
## App Directory

<a name="process-directory"></a>
### Process Directory

The `Process` directory is the default root for all your Process-based containers.

<a name="thread-directory"></a>
### Thread Directory

The `Thread` directory is the default root for all your Thread-based containers.

<a name="data-directory"></a>
## Data Directory

<a name="autorun-directory"></a>
### Autorun Directory

The `autorun` directory is the place where all scripts used by Kraken are stored. You might want to pay special attention to `kraken.process` and `kraken.thread` files inside, which are scripts that runs processes and threads.

<a name="bootstrap-directory"></a>
### Bootstrap Directory

The `bootstrap` directory contains files that bootstrap Kraken runtimes and configure autoloading.

<a name="config-directory"></a>
### Config Directory

The `config` directory, contains all of your runtimes' configuration files. You might want to browse through these files and familiarize yourself with all of the options available to you.

<a name="config-env-directory"></a>
### Config.env Directory

The `config.env` directory, contains your application environment settings.

<a name="log-directory"></a>
### Log Directory

The `log` directory is a place where all of your application logs will be written to.

<a name="storage-directory"></a>
### Storage Directory

The `storage` directory is a place where temporary files and cache will be stored.

<a name="kraken-file"></a>
## Kraken File

The `kraken` file is a script that runs Kraken Console Client.

<a name="kraken-server-file"></a>
## Kraken Server File

The `kraken.server` file is a script that runs Kraken Console Server.

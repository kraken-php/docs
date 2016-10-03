# Installation

- [Installation](#installation)
    - [Server Requirements](#server-requirements)
    - [Installation Process](#installation-process)
    - [Configuration](#configuration)

<a name="installation"></a>
## Installation

<a name="server-requirements"></a>
### Server Requirements

Before installing Kraken Framework make sure your server meets the following requirements:

<div class="dot-list" markdown="1">
- PHP >= 5.5.9,
- Pthreads PHP Extension (only if you want to use threading),
- UNIX or ~~Windows~~ OS.
</div>

<a name="installation-process"></a>
### Installation Process

Kraken uses [Composer](http://getcomposer.org) to manage its dependencies. Before going further in this manual, make sure you have Composer installed on your machine. This can be done by calling command:

    composer --version

#### Installing Skeleton Application

The best way of starting new project is to install Kraken Skeleton Application. It can be done via composer:

    composer create-project --prefer-dist kraken-php/kraken .

After the process is finished, you will find in downloaded package, minimal configuration, empty containers and boot scripts. Everything yout need to start working with Kraken Framework.

<a name="configuration"></a>
### Configuration

#### Run Scripts

There are four important scripts that Framework uses that you should be aware of. Firstly, `kraken` and `kraken.server` being placed in project root are used for starting CLI client and CLI server. Secondly, `kraken.process` and `kraken.thread` files inside `data/autorun` directory are responsible for running processes and threads inside your application. You can modify content of this files to customize provided bootstrap mechanics and change default application paths.

#### Bootstrap Files

Bootstrap files that will be executed before starting each of Kraken runtimes can be found in `data/bootstrap` folder. It consists of three subfolders. `Thread` for keeping common bootstrap logic for all Thread-based runtime containers, `Process` that does the same for Process-based runtime containers and special directory `Console` for both console client and server. When modifying this file it is recommended to make changes only in appointed places.

#### Configuration Files

All of the configuration files are stored in the `data/config` directory. It consists of three subfolders, in similar to `data/bootstrap` fashion, dividing configurations root for Threads, Processes and Console. Each option is documented, so feel free to look through the files and get familiar with the options available to you.

#### Directory Permissions

You may need to configure permissions for some paths. Directories within the `data/log` and the `data/storage` should be writable by your PHP CLI user or the whole framework will not run.

Kraken needs no other configuration out of the box, you are free to tweak [default configuration options](/docs/{{version}}/configuration).

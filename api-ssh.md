# SSH Component

- [Introduction](#introduction)
- [Features](#features)
- [Concepts](#concepts)
- [Examples](#examples)
    - [Executing Commands](#executing-commands)
    - [Writing Files](#writing-files)
    - [Reading Files](#reading-files)
- [Important Classes & Interfaces](#important-classes-and-interfaces)

<a name="introduction"></a>
## Introduction

SSH is a component that provides consistent interface for executing async SSH commands and working with files using SFTP.

<a name="introduction"></a>
## Features

SSH features:

<div class="dot-list" markdown="1">
- OOP abstraction for PHP SSH2 extension
- Support for variety of authorization methods
- Asynchronous SSH2 commands
- Asynchronous operations on files via SFTP
- Kraken Framework compatibility
</div>

<a name="concepts"></a>
## Concepts

This section contains terminology, useful concepts and definitions, that might be helpful to learn to fully understand this component purpose.

### SSH

SSH uses public-key cryptography to authenticate the remote computer and allow it to authenticate the user, if necessary.

<a name="examples"></a>
## Examples

This section contains examples and patterns that can be used with described component.

<a name="executing-commands"></a>
### Executing Commands

    $loop   = new Loop(new SelectLoop);
    $auth   = new SSH2Password($user, $pass);
    $config = new SSH2Config();
    $ssh2   = new SSH2($auth, $config, $loop);
    
    $ssh2->on('connect:shell', function(SSH2DriverInterface $shell) use($ssh2, $loop) {
        echo "# CONNECTED SHELL\n";
    
        $buffer = '';
        $command = $shell->open();
        $command->write('ls -la');
        $command->on('data', function(SSH2ResourceInterface $command, $data) use(&$buffer) {
            $buffer .= $data;
        });
        $command->on('end', function(SSH2ResourceInterface $command) use(&$buffer) {
            echo "# COMMAND RETURNED:\n";
            echo $buffer;
            $command->close();
        });
        $command->on('close', function(SSH2ResourceInterface $command) use($shell) {
            $shell->disconnect();
        });
    });
    
    $ssh2->on('disconnect:shell', function(SSH2DriverInterface $shell) use($ssh2) {
        echo "# DISCONNECTED SHELL\n";
        $ssh2->disconnect();
    });
    
    $ssh2->on('connect', function(SSH2Interface $ssh2) {
        echo "# CONNECTED\n";
        $ssh2->createDriver(SSH2::DRIVER_SHELL)
             ->connect();
    });
    
    $ssh2->on('disconnect', function(SSH2Interface $ssh2) use($loop) {
        echo "# DISCONNECTED\n";
        $loop->stop();
    });
    
    $loop->onTick(function() use($ssh2) {
        $ssh2->connect();
    });
    
    $loop->start();

<a name="writing-files"></a>
### Writing Files

    $loop   = new Loop(new SelectLoop);
    $auth   = new SSH2Password($user, $pass);
    $config = new SSH2Config();
    $ssh2   = new SSH2($auth, $config, $loop);
    
    $ssh2->on('connect:sftp', function(SSH2DriverInterface $sftp) use($loop, $ssh2) {
        echo "# CONNECTED SFTP\n";
    
        $lines = [ "KRAKEN\n", "IS\n", "AWESOME!\n" ];
        $linesPointer = 0;
    
        $file = $sftp->open(__DIR__ . '/_file_write.txt', 'w+');
        $file->write();
        $file->on('drain', function(SSH2ResourceInterface $file) use(&$lines, &$linesPointer) {
            echo "# PART OF THE DATA HAS BEEN WRITTEN\n";
            if ($linesPointer < count($lines)) {
                $file->write($lines[$linesPointer++]);
            }
        });
        $file->on('finish', function(SSH2ResourceInterface $file) {
            echo "# FINISHED WRITING\n";
            $file->close();
        });
        $file->on('close', function(SSH2ResourceInterface $file) use($sftp) {
            echo "# FILE HAS BEEN CLOSED\n";
            $sftp->disconnect();
        });
    });
    
    $ssh2->on('disconnect:sftp', function(SSH2DriverInterface $sftp) use($ssh2) {
        echo "# DISCONNECTED SFTP\n";
        $ssh2->disconnect();
    });
    
    $ssh2->on('connect', function(SSH2Interface $ssh2) {
        echo "# CONNECTED\n";
        $ssh2->createDriver(SSH2::DRIVER_SFTP)
             ->connect();
    });
    
    $ssh2->on('disconnect', function(SSH2Interface $ssh2) use($loop) {
        echo "# DISCONNECTED\n";
        $loop->stop();
    });
    
    $loop->onTick(function() use($ssh2) {
        $ssh2->connect();
    });
    
    $loop->start();

<a name="reading-files"></a>
### Reading Files

    $loop   = new Loop(new SelectLoop);
    $auth   = new SSH2Password($user, $pass);
    $config = new SSH2Config();
    $ssh2   = new SSH2($auth, $config, $loop);
    
    $ssh2->on('connect:sftp', function(SSH2DriverInterface $sftp) use($loop, $ssh2) {
        echo "# CONNECTED SFTP\n";
    
        $buffer = '';
        $file = $sftp->open(__DIR__ . '/_file_read.txt', 'r+');
        $file->read();
        $file->on('data', function(SSH2ResourceInterface $file, $data) use(&$buffer) {
            $buffer .= $data;
        });
        $file->on('end', function(SSH2ResourceInterface $file) use(&$buffer) {
            echo "# FOLLOWING LINES WERE READ FROM FILE:\n";
            echo $buffer;
            $file->close();
        });
        $file->on('close', function(SSH2ResourceInterface $file) use($sftp) {
            echo "# FILE HAS BEEN CLOSED\n";
            $sftp->disconnect();
        });
    });
    
    $ssh2->on('disconnect:sftp', function(SSH2DriverInterface $sftp) use($ssh2) {
        echo "# DISCONNECTED SFTP\n";
        $ssh2->disconnect();
    });
    
    $ssh2->on('connect', function(SSH2Interface $ssh2) {
        echo "# CONNECTED\n";
        $ssh2->createDriver(SSH2::DRIVER_SFTP)
             ->connect();
    });
    
    $ssh2->on('disconnect', function(SSH2Interface $ssh2) use($loop) {
        echo "# DISCONNECTED\n";
        $loop->stop();
    });
    
    $loop->onTick(function() use($ssh2) {
        $ssh2->connect();
    });
    
    $loop->start();

<a name="important-classes-and-interfaces"></a>
## Important Classes & Interfaces

This section contains list of most important classes and interfaces shipped with this component. It **does not include all** classes and interface.

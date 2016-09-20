# Project Deployment

- [Introduction](#introduction)
- [Project Deployment](#project-deployment)
    - [Creating Project](#creating-project)
    - [Checking Status](#checking-status)
    - [Destroying Project](#destroying-project)
- [Getting Help](#getting-help)

<a name="introduction"></a>
## Introduction

Kraken provides your application with powerful console-based [client](/docs/{{version}}/cli#console) and [server](/docs/{{version}}/cli#server) deployment tools. You can read more about them in [CLI section](/docs/{{version}}/cli). 

<a name="project-deployment"></a>
## Project Deployment

To successfuly deploy project using Kraken Framework you have to remember it is necessary to start Kraken Server and keep it running the whole time.

To start server simply run command:

    php kraken.server

You can run it as background process on UNIX system using:

    php kraken.server >/dev/null 2>&1 &

While this method can be used safely for testing and development, you have to keep in mind that Kraken Server is not controlled by Kraken Supervisors and is the only part of architecture that is assumed working properly always. To ensure that this is true, you can use system supervisors.

After starting server, you will be able to use `kraken` script to invoke commands. You can check if connection has been established by using:

    php kraken server:ping

If the returned value is a success, then you are ready to go.

### Supervisors

#### UNIX Supervisors

One of the supervisor tools that can be used together with Kraken Server safely on UNIX systems is [supervisord](http://supervisord.org). Supervisord can be installed with any of the following tools: pip, easy_install, apt-get, yum. Once it is installed you have to generate a template file via following command:

    echo_supervisord_conf > supervisor.conf
    
After the process is complete your should modify the file and configure it properly for your needs. Exemplary configuration that can be used is:

```
[unix_http_server]
file = /tmp/supervisor.sock

[supervisord]
logfile          = ./logs/supervisord.log
logfile_maxbytes = 128MB
logfile_backups  = 10
loglevel         = info
pidfile          = /tmp/supervisord.pid
nodaemon         = false
minfds           = 1024
minprocs         = 200

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl = unix:///tmp/supervisor.sock

[program:ratchet]
command                 = bash -c "exec /usr/bin/php ./kraken.server"
process_name            = Kraken_Server
numprocs                = 1
autostart               = true
autorestart             = true
user                    = root
stdout_logfile          = ./logs/info.log
stdout_logfile_maxbytes = 1MB
stderr_logfile          = ./logs/error.log
stderr_logfile_maxbytes = 1MB
```

<a name="creating-project"></a>
### Creating Project

To start your project run:

    php kraken project:create

<a name="checking-status"></a>
### Checking Status

To check project status run:

    php kraken project:status

<a name="destroying-project"></a>
### Destroying Project

After you are finished and wants to shutdown project, type:

    php kraken project:destroy

<a name="getting-help"></a>
## Getting Help

#### Help About Command

To see help page of any command run:

    php kraken help command

Where `command` is the name of the command you are unsure about, for example:

    php kraken help kraken:create

#### Listing All Commands

To list all commands run:

    php kraken

Aside from the options described in this pages, Kraken provides you with much, much more commands that can be used to manage or adapt distributed application on the fly. To see them all please go to [CLI section](/docs/{{version}}/cli).

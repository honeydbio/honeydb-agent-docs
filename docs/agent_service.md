
# Agent Service

## Linux

By default, the HoneyDB Agent service is configured with systemd. You can use the following commands to control the agent service.

Start the agent:

```shell
# systemctl start honeydb-agent
```

Stop the agent:

```shell
# systemctl stop honeydb-agent
```

Restart the agent:

```shell
# systemctl restart honeydb-agent
```

## Windows

On Windows you can start, stop, or restart the service from the services control panel Alternatively, you can use the following commands:

Start the agent:

```shell
net start honeydb-agent
```

Stop the agent:

```shell
net stop honeydb-agent
```

Restart the agent:

```shell
net restart honeydb-agent
```

!!! note
    To start/stop the service from the command line, you will need to open a command terminal with administrator privileges.

## OSX

tbd

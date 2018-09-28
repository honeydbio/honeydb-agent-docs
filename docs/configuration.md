# Configuring HoneyDB Agent

There are two configuration files for HoneyDB Agent, The default location for both files is the `/etc/honeydb`. The main configuration file is `agent.conc`, and the services configuration file is `services.conf`.

## Agent Configuration

In the `agent.conf` file, the main configuration section is the `[agent]` section and actually only has one option to configure.

Name | Description
---------- | -------
nodename | Name for this agent node to be displayed in tweets (if Twitter is configured).

## Service Configuration

The `service.conf` file tells HoneyDB Agent which services to launch. The service configuration file is used to define service names, ports, and plugins to run on your honeypot. Each service defined in the file has an `enabled` option. This option can be set to Yes or No to determine which services run on start.

The default `services.conf` file comes with all services pre-configured. Review this file to see what plugins are available.

### Services

#### Echo

Echo back data received via tcp.

Plugin name: `Echo`

#### Echo.udp

Echo back data received via udp.

Plugin name: `Echo.udp`

#### FTP

FTP service.

Plugin name: `FTP_tcp`

#### SSH

Documentation to come.

#### Telnet

Documentation to come.

#### SMTP

Documentation to come.

#### DNS.udp

Documentation to come.

#### TFTP

Documentation to come.

#### HTTP

Documentation to come.

#### Modbus

Documentation to come.

#### Random

Documentation to come.

#### Telnet.IoT

Documentation to come.

#### MySQL

Documentation to come.

#### VNC

Documentation to come.

#### Redis

Documentation to come.

#### WebLogic

Documentation to come.

#### Elasticsearch

Documentation to come.

#### Memcached

Documentation to come.

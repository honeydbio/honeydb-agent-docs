# Welcome to HoneyDB Agent Docs!

[![Documentation Status](https://readthedocs.org/projects/honeypy/badge/?version=latest)](http://honeydb-agent-docs.readthedocs.io/en/latest/?badge=latest)

HoneyDB ([honeydb.io](http://honeydb.io)) provides real time data of honeypot activity. This data comes from honeypot sensors deployed globally on the Internet.

**HoneyDB Agent**

The honeydb-agent is a small honeypot program that can be easily deployed on Linux or Windows systems. The honeydb-agent can be configured to particpate in the HoneyDB network or operate standalone.  When running as a standalone agent all activity can be logged to local files, but the primary purpose for the honeydb-agent is to contribute log data back to the [HoneyDB website](http://honeydb.io). The honeydb-agent can implement numerous network service emulations (plugins). The level of interaction is determined by the functionality of the plugins.

When configured to contribute log data to HoneyDB, all data are accessible via the [HoneyDB REST API](https://riskdiscovery.com/honeydb/threats). This means you can deploy the honeydb-agent quickly with minimal configuration effort and no requirement for log management.

# Configuring HoneyDB Agent

There are two configuration files for HoneyDB Agent, The default location for both files is the `/etc/honeydb`. The main configuration file is `agent.conf`, and the services configuration file is `services.conf`.

## Agent Configuration

In the `agent.conf` file, the main configuration section is the `[agent]` section and actually only has one option to configure.

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


# Welcome to HoneyDB Agent Docs!

[![Documentation Status](https://readthedocs.org/projects/honeypy/badge/?version=latest)](http://honeydb-agent-docs.readthedocs.io/en/latest/?badge=latest)

HoneyDB ([honeydb.io](http://honeydb.io)) provides real time data of honeypot activity. This data comes from honeypot sensors deployed globally on the Internet.

## HoneyDB Agent

The honeydb-agent is a small honeypot program that can be easily deployed on Linux or Windows systems. The honeydb-agent can be configured to particpate in the HoneyDB network or operate standalone.  When running as a standalone agent all activity can be logged to local files, but the primary purpose for the honeydb-agent is to contribute log data back to the [HoneyDB website](http://honeydb.io). The honeydb-agent can implement numerous network service emulations (plugins). The level of interaction is determined by the functionality of the plugins.

When configured to contribute log data to HoneyDB, all data are accessible via the [HoneyDB REST API](https://honeydb.io/threats). This means you can deploy the honeydb-agent quickly with minimal configuration effort and no requirement for log management.

## Installation

Install packages are hosted by [Cloudsmith.io](https://cloudsmith.io), current version:

<img src="https://api-prd.cloudsmith.io/badges/version/honeydb/honeydb-agent/deb/honeydb-agent/latest/d=debian%252Fbookworm/?render=true" alt="Latest Version @ Cloudsmith" />

### Linux

On Linux, the honeydb-agent can be installed via your package management system. First, you will need to add the honeydb-agent repository to your system, then it can be installed via package management tools (e.g `apt-get` or `yum`).

Instructions for adding the honeydb-agent package repository to you system can be found on the HoneyDB website. For each operation system there is a Quick Setup and Manual Setup option.

[Amazon Linux / Red Hat / CentOS](https://honeydb.io/downloads#redhat)

[Debian](https://honeydb.io/downloads#debian)

[Ubuntu](https://honeydb.io/downloads#ubuntu)

[Raspbian (ARM)](https://honeydb.io/downloads#raspbian)

### Windows

On Windows the agent is packaged in a `.zip` file. Simply extract the contents of the zip file to a location of your preference. Example `C:\Program Files\honeydb-agent`. To install the agent as a system service, right click on the __Install.bat__ (included in the download package) and select, "Run as administrator".

Download package: [Win32](https://honeydb.io/downloads#windows)

### OSX

On OSX the agent is packaged in a `.tar.gz` file. Extract the contents of the tar.gz file to a temproary location. Open a terminal and change directory to
the termporary location. Then run `./installer install`. The script uses the `sudo` command so you will be prompted for your password.

Example installation steps:

```shell
cd ~/Downloads
tar -xzf honeydb-agent-osx.tar.gz
./install install
```

Download package: [OSX](https://honeydb.io/downloads#osx)

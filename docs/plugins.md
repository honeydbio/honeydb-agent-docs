# Plugins

Plugins are what drive service emulation and determine the level of interaction possible with a given service. Plugins can either be based on TCP or UDP protocols. Plugins also determine what events and data are logged.

## Events

For TCP plugins, all connection (CONNECT) events, data received (RX) events, and data transmitted (TX) events are logged.

For TCP plugins, all data received (RX) events and data transmitted (TX) events are logged. With UDP there are no CONNECT events.

Either TCP or UDP plugins may also log information (INFO) events. These events contain specific data of interest, typically extracted from an RX event. For example, usernames and passwords maybe captured in an INFO event.

## Plugins List

### DNS_udp

A low interaction UDP Domain Name Service (DNS). For each DNS query to this service a random IP address is generated and returned to the client.

Example service configuration:

```shell
[DNS.udp]
plugin      = DNS_udp
low_port    = udp:53
port        = udp:53
description = DNS service.
enabled     = Yes
```

### Echo_tcp

A low interaction TCP echo service. Any data sent to this service will be echoed back to the client.

Example service configuration:

```shell
[Echo]
plugin      = Echo_tcp
low_port    = tcp:7
port        = tcp:7
description = Echo back data received via tcp.
enabled     = Yes
```

### Echo_udp

A low interaction UDP echo service. Any data sent to this service will be echoed back to the client.

Example service configuration:

```shell
[Echo.udp]
plugin      = Echo_udp
low_port    = udp:7
port        = udp:7
description = Echo back data received via udp.
enabled     = Yes
```

### Elasticsearch_tcp

A low interaction TCP Elasticsearch service. This service will handle requests to **/**, **/_nodes** and **/_search**, and will provide fake responses to the client.

### FTP_udp

A low interaction TCP FTP service. This service will handle FTP login attempts but always return invalid login.

Example service configuration:

```shell
[FTP]
plugin      = FTP_tcp
low_port    = tcp:21
port        = tcp:21
description = FTP service.
enabled     = Yes
```

### GAS_tcp

A low interaction TCP gas tank service. This service emulations gas station fuel tanks. It will return randomly generated gas tank metrics back to the client.

Example service configuration:

```shell
```

### HashCountRandom_tcp

A low interaction TCP service. This service will return an MD5 hash and integer to the client. For each client connection, the counter will increment by 1 on each RX event.

Example service configuration:

```shell
```

### HTTP_tcp

A low interaction TCP HTTP service. This service will provide a generic HTTP page to client requests. For certain known targeted applications like phpMyAdmin, wordpress, weblogic, tomcat, and jboss, static responses specific to those applications are returned to the client.

Example service configuration:

```shell
```

### iKettle_tcp

A low interaction TCP smart kettle service. This service will respond to requests containing HELLOKETTLE with HELLOAPP.

Example service configuration:

```shell
```

### LDAP_tcp

A low interaction TCP LDAP service. This service will provide generic responses to LDAP queries.

Example service configuration:

```shell
```

### Memcached_tcp

A low interaction TCP Memcached service. This service will provide generic responses to Memcached queries.

Example service configuration:

```shell
```

### Modbus_tcp

A low interaction TCP Modbus service. This service will provide generic responses to Modbus queries.

Example service configuration:

```shell
```

### MOTD_tcp

A low interaction TCP Message Of The Day (MOTD) service. This service will respond with a generic message.

Example service configuration:

```shell
[MOTD]
plugin      = MOTD_tcp
low_port    = tcp:8
port        = tcp:8
description = Send a message via tcp and close connection.
enabled     = Yes
```

### MOTD_udp

A low interaction UDP Message Of The Day (MOTD) service. This service will respond with a generic message.

Example service configuration:

```shell
[MOTD.udp]
plugin      = MOTD_udp
low_port    = udp:8
port        = udp:8
description = Send a message via udp.
enabled     = Yes
```

### MQTT_tcp

A low interaction TCP MQTT service. This service will provide generic responses to MQTT queries.

Example service configuration:

```shell

```

### MySQL_tcp

A low interaction TCP MySQL service. This service will handle login handshakes, but there is no succesful login/query interaction.

Example service configuration:

```shell
```

### NTP_udp

A low interaction TCP NTP service. This service will provide generic responses to NTP queries.

Example service configuration:

```shell
```

### ProConOs_tcp

A low interaction TCP ProConOs service. This service will provide generic responses to ProConOs queries.

Example service configuration:

```shell
```

### Random_tcp

A low interaction TCP service that responds with random data.

Example service configuration:

```shell
```

### RDP_tcp

A low interaction TCP Remote Desktop Protocol (RDP) service. This service will emulate a Windows login screen and attempts to login as Administrator.

Example service configuration:

```shell
```

### Redis_tcp

A low interaction TCP Redis service. This service will provide generic responses to Redis queries.

Example service configuration:

```shell
```

### SIP_udp

A low interaction UDP SIP service. This service will provide generic responses to SIP queries.

Example service configuration:

```shell
```

### SMTP_tcp

A low interaction UDP SMTP service. This service will provide generic responses to SMTP queries.

Example service configuration:

```shell
[SMTP]
plugin      = SMTP_tcp
low_port    = tcp:25
port        = tcp:25
description = SMTP service.
enabled     = Yes
```

### SNMP_udp

A low interaction UDP SNMP service. This service will provide generic responses to SNMP queries.

Example service configuration:

```shell
```

### SSH_tcp

A low interaction TCP SSH service. This service will handle login handshakes, but there is no succesful login/command interaction.

Example service configuration:

```shell
[SSH]
plugin      = SSH_tcp
low_port    = tcp:22
port        = tcp:22
description = SSH service.
enabled     = Yes
```

### Telnet_tcp

A medium interaction TCP Telnet service. This service will provide generic guessable logins and emulate basic commands post login.

Example service configuration:

```shell
[Telnet]
plugin      = Telnet_tcp
low_port    = tcp:23
port        = tcp:23
description = Telnet service.
enabled     = Yes
```

### TFTP_udp

A medium interaction TCP TFTP service. This service will provide generic responses to TFTP queries.

Example service configuration:

```shell
```

### VNC_tcp

A low interaction TCP VNC service. This service will handle login handshakes, but there is no succesful login interaction.

Example service configuration:

```shell
```

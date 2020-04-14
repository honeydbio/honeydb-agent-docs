# Plugins

Plugins are what drive service emulation and determine the level of interaction is possible with a given service. Plugins can either be based on TCP or UDP protocols.

For TCP plugins, all connection (CONNECT) events, data received (RX) events, and data transmitted (TX) events are logged.

For TCP plugins, all data received (RX) events and data transmitted (TX) events are logged. With UDP there are no CONNECT events.

## DNS_udp

A low interaction UDP Domain Name Service (DNS). For each DNS query to this service a random IP address is generated and returned to the client.

## Echo_tcp

A low interaction TCP echo service. Any data sent to this service will be echoed back to the client.

## Echo_udp

A low interaction UDP echo service. Any data sent to this service will be echoed back to the client.

## Elasticsearch_tcp

A low interaction TCP Elasticsearch service. This service will handle requests to **/**, **/_nodes** and **/_search**, and will provide fake responses to the client.

## FTP_udp

A low interaction TCP FTP service. This service will handle FTP login attempts but always return invalid login.

## GAS_tcp

A low interaction TCP gas tank service. This service emulations gas station fuel tanks. It will return randomly generated gas tank metrics back to the client.

## HashCountRandom_tcp

A low interaction TCP service. This service will return an MD5 hash and integer to the client. For each client connection, the counter will increment by 1 on each RX event.

## HTTP_tcp

A low interaction TCP HTTP service. This service will provide a generic HTTP page to client requests. For certain known targeted applications like phpMyAdmin, wordpress, weblogic, tomcat, and jboss, static responses specific to those applications are returned to the client.

## iKettle_tcp

A low interaction TCP smart kettle service. This service will respond to requests containing HELLOKETTLE with HELLOAPP.

## LDAP_tcp

A low interaction TCP LDAP service. This service will provide generic responses to LDAP queries.

## Memcached_tcp

A low interaction TCP Memcached service. This service will provide generic responses to Memcached queries.

## Modbus_tcp

A low interaction TCP Modbus service. This service will provide generic responses to Modbus queries.

## MOTD_tcp

A low interaction TCP Message Of The Day (MOTD) service. This service will respond with a generic message.

## MOTD_udp

A low interaction UDP Message Of The Day (MOTD) service. This service will respond with a generic message.

## MQTT_tcp

A low interaction TCP MQTT service. This service will provide generic responses to MQTT queries.

## MySQL_tcp

## NTP_udp

## ProConOs_tcp

## Random_tcp

## RDP_tcp

## Redis_tcp

## SIP_udp

## SMTP_tcp

## SNMP_udp

## SSH_tcp

## Telnet_tcp

## TFTP_udp

## VNC_tcp

# Plugins

Plugins are what drive service emulation and determine the level of interaction possible with a given service. Plugins can either be based on TCP or UDP protocols. Plugins also determine what events and data are logged.

## Events

For TCP plugins, all connection (CONNECT) events, data received (RX) events, and data transmitted (TX) events are logged.

For UDP plugins, all data received (RX) events and data transmitted (TX) events are logged. With UDP there are no CONNECT events.

Either TCP or UDP plugins may also log information (INFO) events. These events contain specific data of interest, typically extracted from an RX event. For example, usernames and passwords may be captured in an INFO event.

## Plugin Configuration Files

Many plugins support an optional JSON configuration file that customizes the plugin's behavior. To use one, reference the file from the service definition with the `config` key:

```shell
config      = /etc/honeydb/example.json
```

All plugin configuration keys are optional. If no `config` key is present, the plugin runs with its defaults.

## Plugins List

### ClickhouseHTTP_tcp

A low interaction TCP service that emulates the ClickHouse HTTP interface. It responds to standard ClickHouse endpoints (`/`, `/ping`, `/replicas_status`, `/play`, `/dashboard`, `/metrics`), answers SQL queries with canned responses, and captures credential attempts, queries, and probes of unknown paths.

Example service configuration:

```shell
[ClickhouseHTTP]
plugin      = ClickhouseHTTP_tcp
low_port    = tcp:8123
port        = tcp:8123
description = Clickhouse HTTP interface
enabled     = Yes
config      = /etc/honeydb/clickhouse.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `clickhouse_version` | string | `"24.3.2.23"` | Advertised ClickHouse version string. |
| `clickhouse_server_display_name` | string | `"ch-edge-01"` | Value for the `X-ClickHouse-Server-Display-Name` header. |
| `clickhouse_timezone` | string | `"UTC"` | Value for the `X-ClickHouse-Timezone` header. |
| `auth_policy` | string | `"always_succeed"` | One of `"always_succeed"`, `"always_fail"`, `"match_list"`. |
| `credentials` | list | `[["default", ""], ["admin", "admin"], ["clickhouse", "clickhouse"]]` | List of `[user, password]` pairs used with `auth_policy: match_list`. |
| `require_credentials` | bool | `false` | When `true`, anonymous requests receive a 401 challenge. |
| `accepted_auth_schemes` | list | `["basic","header","query"]` | Subset of auth surfaces to recognize. |
| `enabled_endpoints` | list | `["/", "/ping", "/replicas_status", "/play", "/dashboard", "/metrics", "/js/*"]` | Glob-aware allowlist of paths to serve; others receive 404. |
| `force_close_per_response` | bool | `false` | Close connection after every response. |
| `honor-xff-header` | bool | `false` | Trust the `X-Forwarded-For` header to resolve the effective client IP. |
| `max_request_bytes` | int | `1048576` | Request size cap; larger payloads receive 413. |
| `max_requests_per_connection` | int | `1000` | Force `Connection: close` after this many requests. |
| `idle_timeout_seconds` | int | `60` | Keep-alive idle timeout in seconds. |
| `structured_info_events` | bool | `true` | Emit JSON-encoded INFO payloads; `false` reverts Basic auth events to legacy free-text. |
| `query_indicator_max_bytes` | int | `8192` | Truncation cap for the `query` field in query events. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `http_client` | Captures the User-Agent, Content-Type, method, path, and query string of each HTTP request. |
| `xff_seen` | Fires when an `X-Forwarded-For` header is present; records the peer, XFF value, whether it was honored, and the resolved client. |
| `auth_attempt` | Captures an authentication attempt (auth type, username, password, success). |
| `query` | Captures a received ClickHouse SQL query, its delivery channel and format, and whether a canned response matched. |
| `unknown_path` | Captures a request to an unrecognized HTTP path (method, path, query). |

### CouchDB_tcp

A low interaction TCP service that emulates an Apache CouchDB HTTP server. Supports two version profiles (`1.x` and `3.x`), a realistic database and document API, Basic and cookie authentication, and detection of known CVE exploit patterns (CVE-2017-12635, CVE-2017-12636).

Example service configuration:

```shell
[CouchDB]
plugin      = CouchDB_tcp
low_port    = tcp:5984
port        = tcp:5984
description = CouchDB HTTP interface
enabled     = Yes
config      = /etc/honeydb/couchdb.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `profile` | string | `"1.x"` | Version profile: `"1.x"` or `"3.x"`. |
| `couchdb_version` | string | `"1.6.1"` / `"3.3.3"` | Advertised CouchDB version (default varies by profile). |
| `erlang_version` | string | `"R16B03"` / `"24.3.4.13"` | Erlang OTP version in the `Server` header (default varies by profile). |
| `server_version` | string | — | Overrides the full `Server` header verbatim when set. |
| `vendor_name` | string | `"Ubuntu"` | OS vendor name in the welcome body. |
| `vendor_version` | string | `"22.04"` | OS vendor version in the welcome body. |
| `auth_policy` | string | `"always_succeed"` | One of `"always_succeed"`, `"always_fail"`, `"admin_only"`. |
| `preseed_dbs` | list | profile defaults | Override the list of databases pre-populated at connect time. |
| `max_request_bytes` | int | `1048576` | Request size cap; larger payloads receive 413. |
| `max_dbs_per_connection` | int | `1000` | Maximum databases that may be created in one session. |
| `max_docs_per_db` | int | `1000` | Maximum documents per database per session. |
| `max_requests_per_connection` | int | `1000` | Force connection close after this many requests. |
| `idle_timeout_seconds` | int | `60` | Keep-alive idle timeout in seconds. |
| `tag_known_cves` | bool | `true` | Enable CVE pattern detection events. |
| `structured_info_events` | bool | `true` | Emit JSON-encoded INFO payloads; `false` reverts Basic auth events to legacy free-text. |
| `honor_xff_header` | bool | `false` | Trust the `X-Forwarded-For` header to resolve the effective client IP. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `auth_attempt` | Captures a Basic or cookie/session login attempt (auth type, username, password, success). |
| `cve_attempt` | Fires when a request matches a known CouchDB CVE signature (CVE id, method, path, indicators). |
| `novel_path` | Captures a request to a path outside the known/profiled route set. |
| `db_operation` | Captures a database-level operation (create/delete/info, database name, success). |
| `doc_operation` | Captures a document-level operation (database, document id, success). |

### DNS_tcp

A low interaction TCP Domain Name Service (DNS). Answers queries against a configurable synthetic zone, supports query pipelining on a single connection, detects non-DNS traffic (such as TLS handshakes against the DNS port), and can optionally engage zone transfer (AXFR) requests. Abuse patterns such as DNS tunneling, DGA-like names, and enumeration bursts are detected and logged. Rate limiting state is shared with `DNS_udp`, so an attacker rate-limited on UDP is also rate-limited on TCP.

Example service configuration:

```shell
[DNS.tcp]
plugin      = DNS_tcp
low_port    = tcp:53
port        = tcp:53
description = DNS service.
enabled     = Yes
config      = /etc/honeydb/dns.json
```

#### Config

The configuration schema is identical to `DNS_udp` (see below) plus the TCP-specific keys in this table. Pointing both `[DNS.udp]` and `[DNS.tcp]` at the same JSON file keeps the synthetic zone and detector settings in sync.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `tcp_idle_timeout_seconds` | int | `30` | Connection closes after this many seconds of silence. |
| `max_request_bytes` | int | `65535` | TCP framing cap. Messages whose length prefix exceeds this close the connection. |
| `axfr_policy` | string | `"refused"` | One of `"refused"` or `"engage"`. Engagement streams the full synthetic zone in a multi-message AXFR. |
| `max_axfr_engagements_per_minute` | int | `1` | Per-source cap on AXFR engagements. `0` disables engagement entirely. |
| `max_axfr_messages_per_engagement` | int | `100` | Hard cap on AXFR stream length. Larger zones are truncated. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `dns_query` | Captures each parsed DNS query (qname, qtype, returned rcode). |
| `abuse_pattern` | Fires when a query matches an abuse detector or error condition; a `pattern` field discriminates the type (`parse_error`, `rate_limited`, `internal_error`, `chaos_probe`, amplification probe, zone-transfer attempt, DNSSEC probe, tunneling, DGA-like apex, enumeration burst, and the TCP-only `non_dns_traffic`). |

### DNS_udp

A low interaction UDP Domain Name Service (DNS). Answers queries against a configurable synthetic zone (a default `example.com.` zone is built in), responds to CHAOS-class fingerprint queries (`version.bind`, etc.), and detects abuse patterns such as DNS tunneling, DGA-like names, enumeration bursts, and amplification probes.

Example service configuration:

```shell
[DNS.udp]
plugin      = DNS_udp
low_port    = udp:53
port        = udp:53
description = DNS service.
enabled     = Yes
config      = /etc/honeydb/dns.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `bind_version` | string | `"8.2.2-P5"` | Fills the `version.bind` and `version.server` CHAOS responses when those aren't explicitly set in `chaos`. |
| `chaos` | dict | built-in defaults | Per-CHAOS-name TXT strings (`version.bind`, `version.server`, `hostname.bind`, `id.server`, `authors.bind`). |
| `zones` | list | built-in example.com zone | List of zone definitions (see zone schema below). |
| `out_of_zone_policy` | string | `"nxdomain"` | Response for out-of-zone queries: `"nxdomain"`, `"refused"`, or `"random_a"`. |
| `rate_limit` | dict | `qps=10, burst=20, max_sources=10000` | Per-source-IP rate limit. `qps: 0` disables. Shared across DNS_udp and DNS_tcp. |
| `max_response_bytes_edns` | int | `1232` | Response size cap when the query carries EDNS0. |
| `max_response_bytes_legacy` | int | `512` | Response size cap when the query does not carry EDNS0. |
| `tag_known_abuse` | bool | `true` | Master switch for the abuse-pattern detectors (tunneling, DGA, enumeration, etc.). |
| `tunneling_qname_length_threshold` | int | `80` | Query name length that triggers tunneling detection. |
| `tunneling_entropy_threshold` | float | `3.8` | Label entropy that triggers tunneling detection. |
| `tunneling_high_entropy_min_label_len` | int | `12` | Minimum label length for high-entropy tunneling detection. |
| `dga_entropy_threshold` | float | `3.5` | Entropy threshold for DGA-like name detection. |
| `dga_min_label_len` | int | `10` | Minimum second-level label length for DGA-like name detection. |
| `dga_max_vowel_ratio` | float | `0.15` | Maximum vowel ratio for DGA-like name detection. |
| `enum_burst_threshold` | int | `20` | Distinct child labels under one parent that triggers enumeration detection. |
| `enum_burst_window_seconds` | int | `60` | Sliding window for enumeration tracking. |
| `structured_info_events` | bool | `true` | Set to `false` to revert to the original free-text INFO format. |
| `override_name` | bool | `false` | Set to `true` to report the service name from the config file; otherwise `DNS` is used. |

#### Zone schema

Each entry in `zones` is a dict:

| Key | Type | Description |
|-----|------|-------------|
| `apex` | string | Zone apex, e.g. `"example.com."`. Required. |
| `ttl` | int | Default TTL for records in this zone (default 3600). |
| `soa` | dict | SOA fields: `mname`, `rname`, `serial`, `refresh`, `retry`, `expire`, `minimum`. |
| `ns` | list or dict | NS hostnames. List = apex; dict = `{label: [hostnames]}`. |
| `a`, `aaaa`, `txt`, `cname`, `ptr` | list or dict | List = apex; dict = `{label: [values]}`. |
| `mx` | list or dict | List of `{preference, exchange}` dicts. |
| `srv` | list or dict | List of `{priority, weight, port, target}` dicts. |
| `caa` | list or dict | List of `{flags, tag, value}` dicts. |

Wildcard labels (`*`) are not supported.

#### INFO Events

| Event | Description |
|-------|-------------|
| `dns_query` | Captures each parsed DNS query (qname, qtype, returned rcode). |
| `abuse_pattern` | Fires when a query matches an abuse detector or error condition; a `pattern` field discriminates the type (`parse_error`, `rate_limited`, `internal_error`, `chaos_probe`, amplification probe, zone-transfer attempt, DNSSEC probe, tunneling, DGA-like apex, enumeration burst). |

### Echo_tcp

A low interaction TCP echo service (RFC 862). Any data sent to this service will be echoed back to the client. Received data is also checked for credential-like strings, which are captured as INFO events.

Example service configuration:

```shell
[Echo]
plugin      = Echo_tcp
low_port    = tcp:7
port        = tcp:7
description = Echo back data received via tcp.
enabled     = Yes
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `detect_scanner_fingerprints` | bool | `true` | Enable scanner-attribution INFO events. |
| `detect_credential_like` | bool | `true` | Enable credential-shape INFO events. |
| `event_field_max_bytes` | int | `1024` | Per-string byte truncation cap inside event payloads. |
| `override_name` | bool | `false` | When `false`, the service is reported as `"Echo"` to normalize naming with `Echo_udp`. When `true`, the configured service name is used. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `scanner_fingerprint` | Fires when received bytes match a known scanner fingerprint (scanner name, pattern id, payload size). |
| `credential_like_string` | Fires when received bytes contain credential-like strings. |

### Echo_udp

A low interaction UDP echo service (RFC 862). Data sent to this service is echoed back to the client, capped at `max_echo_bytes` (default 64 bytes) to prevent the service from being abused as a DDoS reflection/amplification vector. Received data is also checked for credential-like strings and amplification probes.

Example service configuration:

```shell
[Echo.udp]
plugin      = Echo_udp
low_port    = udp:7
port        = udp:7
description = Echo back data received via udp.
enabled     = Yes
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `max_echo_bytes` | int | `64` | Maximum bytes reflected back to the source. `0` = never reflect (receive-only). |
| `detect_scanner_fingerprints` | bool | `true` | Enable scanner-attribution INFO events. |
| `detect_credential_like` | bool | `true` | Enable credential-shape INFO events. |
| `detect_amplification_probe` | bool | `true` | Enable amplification-probe INFO events. |
| `amplification_probe_size_threshold` | int | `64` | Datagram size threshold for the oversize amplification indicator. |
| `amplification_probe_repeated_byte_ratio` | float | `0.80` | Repeated-byte ratio threshold for the filler amplification indicator. |
| `event_field_max_bytes` | int | `1024` | Per-string byte truncation cap inside event payloads. |
| `override_name` | bool | `false` | When `false`, the service is reported as `"Echo"` to normalize naming with `Echo_tcp`. |

!!! note
    Earlier versions of this plugin reflected the full received datagram with no size cap. To restore that behavior, set `max_echo_bytes` to a sufficiently large value.

#### INFO Events

| Event | Description |
|-------|-------------|
| `scanner_fingerprint` | Fires when the datagram matches a known scanner fingerprint (scanner name, pattern id, payload size). |
| `credential_like_string` | Fires when the datagram contains credential-like strings. |
| `amplification_probe_suspected` | Fires when a datagram looks like a UDP amplification probe (size / repeated-byte heuristics). |

### Elasticsearch_tcp

A low interaction TCP Elasticsearch service. Emulates the Elasticsearch REST API (root endpoint, `/_nodes`, `/_search`, `/_cat/indices`, `/_cluster/health`, and other commonly probed recon endpoints) with three selectable version profiles. Detects known CVE exploit patterns (CVE-2014-3120, CVE-2015-1427, CVE-2021-44228/Log4Shell).

Example service configuration:

```shell
[Elasticsearch]
plugin      = Elasticsearch_tcp
low_port    = tcp:9200
port        = tcp:9200
description = Send basic elasticsearch replies.
enabled     = Yes
config      = /etc/honeydb/elasticsearch.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `name` | string | `"Flake"` | Elasticsearch node name advertised in responses. |
| `cluster_name` | string | `"elasticsearch"` | Cluster name advertised in responses. |
| `version_profile` | string | `"1.x"` | Version profile: `"1.x"`, `"7.x"`, or `"8.x"`. Controls the response shape for `/` and `/_nodes`. |
| `version_number` | string | profile default | Overrides the `version.number` field. |
| `build_hash` | string | profile default | Overrides the `version.build_hash` field. |
| `build_timestamp` | string | profile default | Overrides the `version.build_timestamp` (or `build_date`) field. |
| `lucene_version` | string | profile default | Overrides the `version.lucene_version` field. |
| `tagline` | string | `"You Know, for Search"` | Tagline in the root response. |
| `tag_known_cves` | bool | `true` | Enable CVE pattern detection events. |
| `honor-xff-header` | bool | `false` | Trust the `X-Forwarded-For` header to resolve the effective client IP. |

Profile defaults:

| Field | `1.x` | `7.x` | `8.x` |
|-------|-------|-------|-------|
| `version_number` | `1.4.1` | `7.17.0` | `8.11.0` |
| `lucene_version` | `4.10.4` | `8.11.1` | `9.7.0` |

#### INFO Events

| Event | Description |
|-------|-------------|
| `user_agent` | Captures the request User-Agent header. |
| `credentials` | Captures HTTP Basic Auth credentials when an Authorization header is present. |
| `novel_path` | Captures a request to a path outside the emulated endpoint set (method, path, query). |
| `body_capture` | Captures the (truncated) request body. |
| `cve_attempt` | Fires on a known CVE pattern (CVE-2014-3120, CVE-2015-1427, Log4Shell CVE-2021-44228); records the CVE id and indicators. |

### FTP_tcp

A low/medium interaction TCP FTP service. Captures credentials and attacker commands, and detects known CVE exploit patterns (CVE-2011-0762 vsftpd backdoor, CVE-2015-3306 ProFTPD mod_copy). At the `low` interaction level all logins are denied; at `medium`, anonymous logins and logins from the password file are allowed, along with file uploads and downloads.

Example service configuration:

```shell
[FTP]
plugin      = FTP_tcp
low_port    = tcp:21
port        = tcp:21
description = FTP service.
enabled     = Yes
config      = /etc/honeydb/ftp.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interaction_level` | string | `"medium"` | `"low"` denies all logins; `"medium"` allows anonymous logins and logins from the password file. |
| `version_profile` | string | `"vsftpd_3"` | Sets the default banner: `vsftpd_2`, `vsftpd_3`, `proftpd`, or `pure_ftpd`. |
| `welcome_message` | string | (profile default) | Overrides the version profile banner when set. |
| `ftp_root` | string | `/tmp/honeydb/ftp` | FTP root directory. This is where FTP user folders are created and file uploads are stored. |
| `password_file` | string | (built-in list) | File containing FTP usernames and passwords in the format `user:password`, one entry per line. |
| `tag_known_cves` | bool | `true` | Enable CVE pattern detection events. |

```
{
    "interaction_level": "medium",
    "version_profile": "vsftpd_3",
    "welcome_message": "(vsFTPd 3.0.3)",
    "ftp_root": "/tmp/honeydb/ftp",
    "password_file": "/etc/honeydb/ftppasswd.dat"
}
```

#### INFO Events

| Event | Description |
|-------|-------------|
| `username` | Captures the username from the USER command. |
| `credentials` | Captures the username/password pair from the PASS command. |
| `cve_attempt` | Fires on a CVE indicator: a USER ending in `:)` (CVE-2011-0762 vsftpd backdoor) or a SITE CPFR/CPTO sequence (CVE-2015-3306 ProFTPD mod_copy). |
| `client` | Captures the reported client version from the CLNT command. |
| `auth_mechanism` | Captures the mechanism requested by the AUTH command. |
| `probe` | Fires on a SYST or FEAT probe command. |
| `port_command` | Captures the parsed IP and port from a PORT command. |
| `eprt_command` | Captures the parsed protocol, address, and port from an EPRT command. |
| `site_command` | Captures a SITE subcommand and its parameters. |
| `directory_traversal` | Fires when a RETR/STOR/LIST/MKD/DELE path contains `../` or `..\`. |

### Gas_tcp

A low interaction TCP gas tank service. Emulates a Veeder-Root TLS Automatic Tank Gauge (ATG) — the industrial control system used at gas stations to monitor underground fuel storage tanks. Responds to common read and write commands with realistic sensor data, and captures attempts to change tank labels or the system date/time.

Example service configuration:

```shell
[Gas]
plugin      = Gas_tcp
low_port    = tcp:10001
port        = tcp:10001
description = Gas pump protocol.
enabled     = Yes
config      = /etc/honeydb/gas.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `station_name` | string | random from built-in list | Fixed station name; overrides random selection. |
| `product1` | string | `"SUPER"` | Tank 1 product label. |
| `product2` | string | `"UNLEAD"` | Tank 2 product label. |
| `product3` | string | `"DIESEL"` | Tank 3 product label. |
| `product4` | string | `"PREMIUM"` | Tank 4 product label. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `protocol_violation` | Fires when a received ATG command is malformed (missing the SOH start byte or too short to be valid). |
| `atg_command` | Captures a recognized ATG command and its type, including date/time-set (S001) and tank/label-set (S602xx) write attempts; unrecognized commands are captured with their raw bytes. |

### HashCountRandom_tcp

A low interaction TCP service. This service will return an MD5 hash and integer to the client. For each client connection, the counter will increment by 1 on each RX event.

Example service configuration:

```shell
[HashCountRandom]
plugin      = HashCountRandom_tcp
low_port    = tcp:4096
port        = tcp:4096
description = Send random data prefixed with a hash of a counter via tcp.
enabled     = Yes
```

### HTTP_tcp

A low interaction TCP HTTP service. This service will provide a generic HTTP page to client requests. For certain known targeted applications like phpMyAdmin, wordpress, weblogic, tomcat, and jboss, static responses specific to those applications are returned to the client.

Example service configuration:

```shell
[HTTP]
plugin      = HTTP_tcp
low_port    = tcp:80
port        = tcp:80
description = HTTP service.
enabled     = Yes
config      = 
```

#### INFO Events

| Event | Description |
|-------|-------------|
| `fingerprint` | Captures the client fingerprint submitted to the JS-based `/session` endpoint. |
| `credentials` | Captures username/password from a POST login form body for a matched target. |
| `client_headers` | Captures the User-Agent and/or Referer sent to a matched target path. |
| `basic_auth` | Captures HTTP Basic Auth credentials for a target. |
| `path_cves` | Captures CVE identifiers associated with a matched configured path. |
| `cve_pattern` | Fires on a CVE pattern match in the request path, body, or headers. |
| `tbk_dvr_command` | Captures the injected command for CVE-2024-3721 (TBK DVR command injection). |
| `tbk_dvr_dropper` | Captures dropper indicators (tools, URLs, hosts, filenames) extracted from the injected command. |
| `tbk_dvr_signatures` | Captures matched loader signatures (user-agent, cookie, magic string) for CVE-2024-3721. |
| `unknown_path` | Captures a request to an unmatched (404) path (method, path, query, User-Agent). |

### iKettle_tcp

A low interaction TCP smart kettle service. Emulates the Smarter iKettle 2.0 smart kettle, which communicates over plain TCP with no authentication and is actively indexed by Shodan and probed by automated scanners. Captures which commands attackers issue — device fingerprinting, control attempts (boil, stop, set temperature), and supplied parameters.

Example service configuration:

```shell
[iKettle]
plugin      = iKettle_tcp
low_port    = tcp:2081
port        = tcp:2081
description = iKettle smart kettle emulation.
enabled     = Yes
```

#### INFO Events

| Event | Description |
|-------|-------------|
| `version_query` | Fires on a firmware/version query command (`0x11`). |
| `status_query` | Fires on a status query command (`0x21`). |
| `stop_command` | Fires on a stop command (`0x61`). |
| `boil_command` | Fires on a boil command (`0x65`); captures the target and keep-warm temperatures. |
| `set_temperature` | Fires on a set-temperature command (`0x69`); captures the target temperature. |
| `unknown_command` | Fires on an unrecognized command byte. |

### IMAP_tcp

A low interaction TCP IMAP service. Emulates an IMAP4rev1 server to attract credential-stuffing attacks, automated mail-client probes, and post-authentication enumeration activity. Captures login credentials, mailbox selections, search criteria, and appended messages.

Example service configuration:

```shell
[IMAP]
plugin      = IMAP_tcp
low_port    = tcp:143
port        = tcp:143
description = IMAP service.
enabled     = Yes
config      = /etc/honeydb/imap.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `server_name` | string | `"localhost"` | Hostname shown in the IMAP greeting and STATUS responses. |
| `version_string` | string | `"Dovecot ready"` | Banner string appended to the greeting. |
| `valid_passwords` | list | `["admin","letmein","password123","password"]` | Passwords accepted for any username. |
| `capabilities_pre_auth` | list | `["IMAP4rev1","LOGIN"]` | Capabilities advertised before authentication. |
| `capabilities_post_auth` | list | `["IMAP4rev1","IDLE","NAMESPACE"]` | Capabilities advertised after authentication. |
| `mailboxes` | list | `["INBOX","Sent","Drafts","Trash","Spam"]` | Mailboxes available to clients. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `credentials` | Captures the username/password from a LOGIN command. |
| `imap_select` | Fires on SELECT or EXAMINE; records the mailbox and read-only flag. |
| `imap_search` | Captures the criteria from a SEARCH command. |
| `imap_append` | Captures the mailbox and uploaded message body from an APPEND command. |

### IPP_tcp

A low interaction TCP Internet Printing Protocol (IPP/2.0) service. Accepts both HTTP-wrapped IPP requests (as used by CUPS, Windows, and exploit frameworks) and raw IPP binary. Captures print-job metadata, usernames, targeted printer URIs, and CVE exploit patterns (CVE-2024-47176, CVE-2024-47177).

Example service configuration:

```shell
[IPP]
plugin      = IPP_tcp
low_port    = tcp:631
port        = tcp:631
description = Internet Printing Protocol service.
enabled     = Yes
config      = /etc/honeydb/ipp.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `upload_dir` | string | `"var/uploads/print_jobs"` | Directory where captured print job files are saved. |
| `save_print_jobs` | bool | `true` | Set to `false` to disable saving print job files while still emitting INFO events. |
| `printer_name` | string | `"HP LaserJet Pro MFP"` | Printer name returned in Get-Printer-Attributes responses. |
| `printer_location` | string | `"Office - Floor 2"` | Location string returned in Get-Printer-Attributes responses. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `ipp_username` | Captures the requesting-user-name attribute and the operation. |
| `ipp_abuse_pattern` | Fires on a shell-injection (CVE-2024-47177) or suspicious printer-uri (CVE-2024-47176) pattern. |
| `ipp_print_job` | Fires on a Print-Job operation. |
| `ipp_validate_job` | Fires on a Validate-Job operation. |
| `ipp_create_job` | Fires on a Create-Job operation. |
| `ipp_send_document` | Fires on a Send-Document operation. |
| `ipp_cancel_job` | Fires on a Cancel-Job operation; records the job id. |

### Kubernetes_tcp

A low interaction TCP Kubernetes service. Emulates a Kubernetes API server, responding to common `kubectl` API paths (nodes, pods, secrets, namespaces, deployments, RBAC) with plausible JSON payloads. Captures bearer tokens, anonymous access, request bodies, and pod exec/attach/portforward attempts.

Example service configuration:

```shell
[Kubernetes]
plugin      = Kubernetes_tcp
low_port    = tcp:6443
port        = tcp:6443
description = Send basic Kubernetes server replies.
enabled     = Yes
```

#### INFO Events

| Event | Description |
|-------|-------------|
| `http_request` | Captures each API request's method, path, and query string. |
| `user_agent` | Captures the request User-Agent header. |
| `bearer_token` | Captures a Bearer token presented in the Authorization header. |
| `anonymous_access` | Fires on a request made with no authentication credentials. |
| `body_capture` | Captures the (truncated) request body. |
| `exec_attempt` | Fires on a pod exec/attach/portforward subresource request. |

### LDAP_tcp

A low interaction TCP LDAP service. Captures bind attempts (anonymous and credential), directory search queries, and write-operation attempts. Detects Log4Shell JNDI injection (CVE-2021-44228) and classic LDAP filter-injection patterns. Binds using any password from a built-in common-password list are accepted; all other passwords are rejected.

Example service configuration:

```shell
[LDAP]
plugin      = LDAP_tcp
low_port    = tcp:389
port        = tcp:389
description = LDAP server
enabled     = Yes
```

#### INFO Events

| Event | Description |
|-------|-------------|
| `cve_attempt` | Fires on a Log4Shell JNDI pattern (CVE-2021-44228) detected in request text. |
| `injection_attempt` | Fires on an LDAP filter-injection pattern detected in request text. |
| `anonymous_bind` | Fires on an anonymous (empty-credential) bind. |
| `bind_attempt` | Captures an authenticated bind attempt and supplied credentials. |
| `search_query` | Captures an LDAP search request. |
| `modify_attempt` | Captures an LDAP modify request. |
| `add_attempt` | Captures an LDAP add request. |
| `delete_attempt` | Captures an LDAP delete request. |

### LPD_tcp

A low interaction TCP Line Printer Daemon (LPD) service (RFC 1179). LPD is a legacy print-spooling protocol commonly found on embedded network printers, older UNIX hosts, and industrial print servers. Captures submitted print jobs, including job metadata (hostname, user, job name) and print data format.

Example service configuration:

```shell
[LPD]
plugin      = LPD_tcp
low_port    = tcp:515
port        = tcp:515
description = LPD service.
enabled     = Yes
config      = /etc/honeydb/lpd.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `save_jobs` | bool | `false` | Write received print job files to disk. |
| `upload_dir` | string | `/tmp/honeydb/var/uploads/print_jobs` | Directory for saved print job files and metadata. Only used when `save_jobs` is `true`. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `job_info` | Captures print-job metadata from the control file (hostname, user, job name, source filename, queue). |
| `data_info` | Captures the print-job data summary (detected print format and data size). |
| `unexpected_queue` | Fires when a job targets a print queue name not in the known-queues list. |

### Memcached_tcp

A low interaction TCP Memcached service. Accepts connections and returns plausible Memcached text-protocol responses without storing any data. Captures which cache keys clients probe, write attempts, and destructive commands such as `flush_all`.

Example service configuration:

```shell
[Memcached]
plugin      = Memcached_tcp
low_port    = tcp:11211
port        = tcp:11211
description = Send basic memcached replies.
enabled     = Yes
config      = /etc/honeydb/memcached.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `version` | string | `"1.6.12"` | Memcached version string returned in `VERSION` and `STATS` responses. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `memcached_keys_probed` | Captures a read command (get/gets) and the keys requested. |
| `memcached_write_attempt` | Captures a write command (set/add/replace/append/prepend) and its key/storage metadata. |
| `memcached_flush` | Captures a `flush_all` command and its optional delay. |

### Modbus_tcp

A low interaction TCP Modbus service. Emulates a Modbus TCP PLC with selectable vendor profiles, handling the common read/write function codes and device identification requests. Captures register/coil read and write attempts and detects scanning behavior (sequential address walks, multi-function-code enumeration).

Example service configuration:

```shell
[Modbus]
plugin      = Modbus_tcp
low_port    = tcp:502
port        = tcp:502
description = Modbus service.
enabled     = Yes
config      = /etc/honeydb/modbus.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `vendor_profile` | string | `"abb"` | Profile key: `abb`, `schneider`, `siemens`, `allen_bradley`, or `generic`. |
| `vendor_name` | string | (from profile) | Override vendor name from profile. |
| `product_code` | string | (from profile) | Override product code from profile. |
| `product_name` | string | (from profile) | Override product name from profile. |
| `model_name` | string | (from profile) | Override model name from profile. |
| `major_minor_revision` | string | (from profile) | Override firmware revision from profile. |
| `vendor_url` | string | (from profile) | Override vendor URL from profile. |
| `user_application_name` | string | (empty) | Override user application name from profile. |
| `holding_registers` | object | — | Map of address string → value string to pre-set holding register values. |
| `input_registers` | object | — | Map of address string → value string to pre-set input register values. |
| `coils` | object | — | Map of address string → value string (0 or 1) to pre-set coil values. |
| `discrete_inputs` | object | — | Map of address string → value string (0 or 1) to pre-set discrete input values. |

Vendor profiles:

| Profile Key | Vendor | Product | Revision |
|-------------|--------|---------|----------|
| `abb` | ABB | AC500 | 3.0.2 |
| `schneider` | Schneider Electric | Modicon M340 | 2.6.1 |
| `siemens` | Siemens | SIMATIC S7-1200 | 4.4.0 |
| `allen_bradley` | Rockwell Automation | MicroLogix 1400 | 21.003 |
| `generic` | Generic PLC | Generic PLC | 1.0.0 |

#### INFO Events

| Event | Description |
|-------|-------------|
| `function_code` | Captures a valid Modbus function-code request (FC, name, unit/transaction IDs, request parameters). |
| `write_attempt` | Captures a write function code that modifies the simulated data store. |
| `scan_pattern` | Fires on detected scanning behavior (sequential address walk or multi-function-code enumeration). |
| `parse_error` | Fires when a frame fails to parse; captures the raw hex of the offending PDU. |

### MOTD_tcp

A low interaction TCP Message Of The Day (MOTD) service. On connect, sends a banner selected at random from the configured list (or built-in IoT/embedded-device defaults: OpenWrt router, Hikvision IP camera, Synology NAS, Home Assistant hub) and closes the connection.

Example service configuration:

```shell
[MOTD]
plugin      = MOTD_tcp
low_port    = tcp:8
port        = tcp:8
description = Send a message via tcp and close connection.
enabled     = Yes
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `banners` | list | built-in IoT list | Banner strings to send on connect. One is selected at random per session. |
| `detect_scanner_fingerprints` | bool | `true` | Enable scanner-attribution INFO events. |
| `detect_credential_like` | bool | `true` | Enable credential-shape INFO events. |
| `event_field_max_bytes` | int | `1024` | Per-string byte truncation cap inside event payloads. |
| `override_name` | bool | `false` | When `false`, the service is reported as `"MOTD"` to normalize naming with `MOTD_udp`. |

```
{
  "banners": [
    "My Custom Device v1.0\r\n",
    "Another Device Banner\r\n"
  ]
}
```

#### INFO Events

| Event | Description |
|-------|-------------|
| `scanner_fingerprint` | Fires when received bytes match a known scanner fingerprint. |
| `credential_like_string` | Fires when received bytes contain credential-like strings. |

### MOTD_udp

A low interaction UDP Message Of The Day (MOTD) service. Replies to each received datagram with a banner selected at random from the configured list (or built-in IoT/embedded-device defaults: OpenWrt router, Hikvision IP camera, Synology NAS, Home Assistant hub).

Example service configuration:

```shell
[MOTD.udp]
plugin      = MOTD_udp
low_port    = udp:8
port        = udp:8
description = Send a message via udp.
enabled     = Yes
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `banners` | list | built-in IoT list | Banner strings to send in reply. One is selected at random per datagram. |
| `detect_scanner_fingerprints` | bool | `true` | Enable scanner-attribution INFO events. |
| `detect_credential_like` | bool | `true` | Enable credential-shape INFO events. |
| `event_field_max_bytes` | int | `1024` | Per-string byte truncation cap inside event payloads. |
| `override_name` | bool | `false` | When `false`, the service is reported as `"MOTD"` to normalize naming with `MOTD_tcp`. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `scanner_fingerprint` | Fires when received datagram bytes match a known scanner fingerprint. |
| `credential_like_string` | Fires when received datagram bytes contain credential-like strings. |

### MQTT_tcp

A low interaction TCP MQTT service. Emulates an MQTT broker that accepts all connections, capturing credentials, topic subscriptions, published payloads, and known IoT device patterns (Home Assistant, Tasmota, Zigbee2MQTT, Shelly, and others) from connecting clients.

Example service configuration:

```shell
[MQTT]
plugin      = MQTT_tcp
low_port    = tcp:1883
port        = tcp:1883
description = MQTT
enabled     = Yes
```

#### INFO Events

| Event | Description |
|-------|-------------|
| `connect` | Captures a CONNECT packet (client ID, protocol version, supplied/absent credentials). |
| `will` | Captures a declared Last Will (topic, message size/preview). |
| `subscribe` | Captures a SUBSCRIBE (topic, wildcard flag, known IoT topic pattern). |
| `publish` | Captures a PUBLISH (topic, payload size/preview, known IoT topic pattern). |
| `large_message` | Fires when a publish message exceeds the logging cap (total/logged/dropped bytes). |
| `disconnect` | Fires on a DISCONNECT packet. |

### MSSQL_tcp

A low interaction TCP Microsoft SQL Server service using the TDS protocol. Handles client PRELOGIN and LOGIN7 handshakes and captures SQL and Windows (NTLM) credentials. When configured with `allow_login: true`, it accepts a fake login and also captures post-authentication SQL queries, stored procedure calls, attack patterns (such as `xp_cmdshell` usage), and known attack tool fingerprints.

Example service configuration:

```shell
[MSSQL]
plugin      = MSSQL_tcp
low_port    = tcp:1433
port        = tcp:1433
description = MSSQL server
enabled     = Yes
config      = /etc/honeydb/mssql.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `mssql_version` | string | `"0f000000"` | Single SQL Server version hex string advertised in PRELOGIN. Ignored if `mssql_versions` is set. |
| `mssql_versions` | list | (built-in pool) | Pool of version strings; one is selected at random per connection. The built-in pool covers SQL Server 2012–2022. |
| `server_name` | string | `"CustomerDB"` | Server name returned in TDS error tokens. |
| `allow_login` | bool | `false` | If `true`, all login attempts are granted a fake successful login and post-auth queries are captured. |

Version hex string reference:

| SQL Server Version | Hex string |
|--------------------|-----------|
| 2012 (v11) | `0b000000` |
| 2014 (v12) | `0c000000` |
| 2016 (v13) | `0d000000` |
| 2017 (v14) | `0e000000` |
| 2019 (v15) | `0f000000` |
| 2022 (v16) | `10000000` |

#### INFO Events

| Event | Description |
|-------|-------------|
| `prelogin` | Captures a parsed PRELOGIN packet and the client's pre-login options. |
| `advertised_version` | Records the server version advertised back during pre-login. |
| `client_login` | Captures a parsed LOGIN7 packet (login details, auth type, result). |
| `ntlm_auth` | Captures NTLM authentication data (domain, username, workstation) from a Windows-auth login. |
| `query` | Captures the decoded SQL text of a client batch query. |
| `rpc` | Captures a parsed RPC request (procedure name, first string parameter). |
| `attack_pattern` | Fires when query/RPC text matches an attack pattern (`xp_cmdshell`, `sp_configure`, UNC injection, enumeration). |
| `tool_fingerprint` | Fires when the client app/library name matches a known tool fingerprint. |
| `parse_error` | Fires when a TDS header or packet fails to parse. |

### MySQL_tcp

A low interaction TCP MySQL service. Emulates a MySQL 5.7/8.x server and simulates a complete authenticated session — every login is accepted — to capture credentials, post-login SQL queries, attack patterns (file read/write, UDF code execution, schema enumeration), and attack tool fingerprints (sqlmap, metasploit, hydra, and others).

Example service configuration:

```shell
[MySQL]
plugin      = MySQL_tcp
low_port    = tcp:3306
port        = tcp:3306
description = MySQL server
enabled     = Yes
config      = /etc/honeydb/mysql.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `mysql_version` | string | (uses pool) | Single version string to advertise in the server greeting. |
| `mysql_versions` | list | built-in pool | Pool of version strings to randomize from per connection — takes precedence over `mysql_version`. |
| `os_version` | string | `"ubuntu0.20.04.3"` | OS suffix appended to the version string in the greeting. |

```
{
    "mysql_versions": ["8.0.32", "8.0.36", "5.7.44-log"],
    "os_version": "ubuntu0.22.04.1"
}
```

#### INFO Events

| Event | Description |
|-------|-------------|
| `client_info` | Captures client handshake-response info during login. |
| `credentials` | Captures the username and auth data from the client login. |
| `query` | Captures the scrubbed SQL text of a COM_QUERY. |
| `db_change` | Captures a database switch (COM_INIT_DB) and the target database. |
| `unknown_command` | Fires on an unrecognized MySQL command byte. |
| `attack_pattern` | Fires when query text matches a known attack pattern. |
| `tool_fingerprint` | Fires when a client name or query matches a known tool fingerprint. |
| `parse_error` | Fires when a packet fails to parse. |

### NTP_udp

A low interaction UDP NTP service. Emulates a stratum-2 NTP server, responding to standard NTP client requests. Detects NTP Mode 7 monlist requests associated with the CVE-2013-5211 NTP amplification DDoS attack.

Example service configuration:

```shell
[NTP]
plugin      = NTP_udp
low_port    = udp:123
port        = udp:123
description = NTP service.
enabled     = Yes
```

#### INFO Events

| Event | Description |
|-------|-------------|
| `ntp_mode7` | Fires on an NTP Mode 7 private-mode packet; tags CVE-2013-5211 on monlist request codes. |
| `ntp_client_info` | Captures a parsed standard NTP request (client version, mode, stratum). |
| `ntp_error` | Fires when packet parsing raises an error. |

### Oracle_tcp

A low interaction TCP Oracle Database service. Emulates an Oracle TNS listener, capturing connect strings, client versions, privilege escalation attempts (`AS SYSDBA`/`AS SYSOPER`), and login credentials. When `allow_login` is enabled, it simulates a successful login and captures post-authentication SQL and attack patterns. Detects CVE-2012-1675 (TNS Poison) indicators.

Example service configuration:

```shell
[Oracle]
plugin      = Oracle_tcp
low_port    = tcp:1521
port        = tcp:1521
description = Oracle TNS listener
enabled     = Yes
config      = /etc/honeydb/oracle.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `oracle_version` | string | `"19.0.0.0.0"` | Oracle version string included in the banner and ACCEPT packet. |
| `oracle_os` | string | `"Linux"` | OS label in the banner string (`Linux` or `Windows`). |
| `allow_login` | bool | `true` | When `true`, simulate a successful login and capture post-auth activity. When `false`, reject with ORA-01017 immediately after credentials are captured. |
| `timeout` | int | `30` | Seconds before closing a connection that has not completed authentication. |
| `post_auth_timeout` | int | `30` | Seconds of inactivity after authentication before closing. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `connect_info` | Captures a TNS CONNECT (service name, client id, username, raw connect string). |
| `cve` | Fires when the service name matches CVE-2012-1675 (TNS Poison) indicators. |
| `privilege_escalation` | Fires when a connect string requests SYSDBA or SYSOPER privileges. |
| `client_version` | Captures the decoded client version when present in the connect string. |
| `auth_attempt` | Captures the submitted username and password. |
| `post_auth_command` | Captures a post-auth SQL statement (type, raw SQL, tables, columns). |
| `attack_pattern` | Fires when post-auth SQL matches a known Oracle attack pattern. |
| `post_auth_summary` | Reports the count of post-auth packets seen at timeout or connection close. |

### POP3_tcp

A low interaction TCP POP3 service. Attracts credential-stuffing attacks, automated mail-client probes, and post-authentication enumeration activity. Captures usernames, passwords, and message retrieval activity.

Example service configuration:

```shell
[POP3]
plugin      = POP3_tcp
low_port    = tcp:110
port        = tcp:110
description = POP3 service.
enabled     = Yes
config      = /etc/honeydb/pop3.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `server_name` | string | `"localhost"` | Hostname shown in the POP3 greeting. |
| `version_string` | string | `"POP3 server ready"` | Banner string appended to the greeting. |
| `valid_passwords` | list | `["admin","letmein","password123","password"]` | Passwords accepted for any username. |
| `capabilities` | list | `["USER","UIDL","TOP"]` | Capabilities advertised by the CAPA command. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `username` | Captures the username from the USER command. |
| `credentials` | Captures the username/password pair from the PASS command. |
| `pop3_retr` | Captures the message number from a RETR command. |
| `pop3_uidl` | Captures the message number from a UIDL command. |
| `pop3_top` | Captures the message number and requested line count from a TOP command. |

### PostgreSQL_tcp

A low interaction TCP PostgreSQL service (wire protocol v3). Captures credential spray attempts, client tool fingerprints (hydra, nmap, sqlmap, metasploit, and others), and — when `allow_login` is enabled — post-authentication queries and attack patterns, including CVE-2019-9193 (`COPY ... FROM PROGRAM`) attempts.

Example service configuration:

```shell
[PostgreSQL]
plugin      = PostgreSQL_tcp
low_port    = tcp:5432
port        = tcp:5432
description = PostgreSQL server
enabled     = Yes
config      = /etc/honeydb/postgresql.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `pg_version` | string | `"14.2"` | PostgreSQL version string advertised after login and embedded in auth failure messages. |
| `os_version` | string | `"14.2-1.pgdg20.04+1"` | OS suffix appended to the version string as `(Ubuntu <os_version>)`. Leave empty to omit. |
| `max_connections` | int | `100` | Advertised after successful auth (allow_login mode). |
| `allow_login` | bool | `false` | When `true`, authentication succeeds (fake) and the session advances to query capture mode. When `false`, authentication always fails after capturing credentials. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `startup_info` | Captures a valid startup message (user, database, encoding, application_name, protocol version). |
| `tool_detection` | Fires when the application_name matches a known client/scanner fingerprint. |
| `auth_attempt` | Captures a password message (user, database, submitted password). |
| `query` | Captures the SQL text of a simple Query message. |
| `attack_pattern` | Fires when query SQL matches a detected attack pattern. |
| `unexpected_message` | Fires when a non-password message arrives during the auth phase. |
| `parse_error` | Fires on an oversized or malformed startup/password/query message. |

### ProConOs_tcp

A low interaction TCP ProConOs service. Emulates a Phoenix Contact ProConOs PLC runtime — the environment used by Phoenix Contact Inline, AXC, and RFC series PLCs — with selectable device profiles. Captures device identification probes, project info queries, and high-severity PLC start/stop control attempts.

Example service configuration:

```shell
[ProConOs]
plugin      = ProConOs_tcp
low_port    = tcp:20547
port        = tcp:20547
description = ProConOs PLC runtime service.
enabled     = Yes
config      = /etc/honeydb/proconos.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `device_profile` | string | `"il_2050_me"` | Profile key; selects all identity strings as a group. One of `il_2050_me`, `axc_f_2152`, `rfc_460r`, `generic`. |
| `ladder_runtime` | string | (from profile) | ProConOs runtime version string returned in the identify response. |
| `plc_type` | string | (from profile) | PLC model and firmware string returned in the identify response. |
| `project_name` | string | (from profile) | Project name returned in identify and project info responses. |
| `boot_project` | string | (from profile) | Boot project name returned in identify and project info responses. |
| `project_source` | string | `"n/a"` | Project source returned in identify and project info responses. |
| `plc_state` | string | (from profile) | PLC operating state (`RUN`, `STOP`, or `HALT`). |
| `probe_threshold` | int | `3` | Number of commands in one session before a scan-pattern INFO event fires. |

Device profiles:

| Profile Key | PLC Type | Project | State |
|-------------|----------|---------|-------|
| `il_2050_me` | IL 2050 ME V 3.95A.6 | QuickMix | RUN |
| `axc_f_2152` | AXC F 2152 V 2.0.4 | MainProcess | RUN |
| `rfc_460r` | RFC 460R PN 3TX V 5.20 | WaterControl | STOP |
| `generic` | Generic PLC V 1.0 | Project | RUN |

#### INFO Events

| Event | Description |
|-------|-------------|
| `identify` | Fires on an IDENTIFY (device fingerprint) request. |
| `plc_state_query` | Fires on a PLC state query. |
| `project_info` | Fires on a project-info request. |
| `write_attempt` | Fires on a PLC start/stop control command (high severity). |
| `unknown_command` | Fires on an unrecognized command byte; captures the byte and raw hex. |
| `scan_pattern` | Fires when the per-session command count reaches the probe threshold. |
| `parse_error` | Fires on a buffer-overflow or frame-dispatch error; captures the raw hex. |

### Random_tcp

A low interaction TCP service that responds with random data. Responds to any inbound connection with a burst of random base64-encoded data, and keeps the connection open — replying to each client payload with a fresh random blob — to engage slow/persistent probers until the client disconnects or the per-session exchange limit is reached.

Example service configuration:

```shell
[Random]
plugin      = Random_tcp
low_port    = tcp:2048
port        = tcp:2048
description = Send random data via tcp.
enabled     = Yes
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `max_exchanges` | int | `100` | Maximum number of exchanges per session. When reached, the server sends one final random response and closes the connection. |

### RAW_tcp

A low interaction TCP JetDirect (RAW printing) service. Captures print jobs sent to the printer port, detects the print data format (PDF, PostScript, PCL, PJL, and others), extracts PJL job metadata, and detects abuse patterns and CVE exploit indicators (CVE-2024-47076, CVE-2024-47177).

Example service configuration:

```shell
[RAW]
plugin      = RAW_tcp
low_port    = tcp:9100
port        = tcp:9100
description = JetDirect raw printing service.
enabled     = Yes
config      = /etc/honeydb/raw.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `upload_dir` | string | `"var/uploads/print_jobs"` | Directory where captured print job files and metadata are saved. |
| `banner` | string | (none) | Bytes sent to the client on connect. Preset values: `"hp_laserjet"`, `"brother"`, `"xerox"`. Any other non-empty string is sent verbatim. Omit or set to empty to send nothing (default). |
| `tag_known_cves` | bool | `true` | Enable CVE pattern detection events. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `print_format` | Captures the detected print format of each print job (PJL, PostScript, PCL, etc.). |
| `pjl_job_name` | Captures the job name from a PJL JOB NAME directive. |
| `pjl_username` | Captures the username from a PJL JOB/SET directive. |
| `pjl_info_probe` | Fires on a PJL INFO/ECHO probe directive. |
| `pjl_filesystem_access` | Fires on a PJL filesystem-access command. |
| `postscript_dangerous_operator` | Fires when a dangerous PostScript operator appears in a job. |
| `cve_attempt` | Fires on path traversal in a PJL FS command (CVE-2024-47076) or Foomatic RIP shell injection in a PJL directive (CVE-2024-47177). |

### RDP_tcp

A low interaction TCP RDP service. Emulates the RDP connection handshake (X.224, TLS upgrade, MCS channel negotiation) to maximize credential capture, then terminates the connection after the client submits credentials. Captures requested security protocols, routing cookies, and submitted domain/username/password values. Detects CVE-2019-0708 (BlueKeep) exploit attempts.

Example service configuration:

```shell
[RDP]
plugin      = RDP_tcp
low_port    = tcp:3389
port        = tcp:3389
description = RDP service.
enabled     = Yes
```

#### INFO Events

| Event | Description |
|-------|-------------|
| `cookie` | Captures the mstshash routing-cookie identifier when present. |
| `requested_protocols` | Captures the requested security protocols from the RDP negotiation request. |
| `cve` | Fires when BlueKeep (CVE-2019-0708) ms_t120 channel binding is detected. |
| `credentials` | Captures the domain, username, and password from the Info PDU. |

### Redis_tcp

A low interaction TCP Redis service. Emulates a Redis 7.2.4 server, capturing authentication attempts, configuration changes, Lua script execution, replication commands, and module load attempts. Detects known Redis attack patterns including CVE-2022-0543 (Lua sandbox escape), config-based RCE, and module-based RCE.

Example service configuration:

```shell
[Redis]
plugin      = Redis_tcp
low_port    = tcp:6379
port        = tcp:6379
description = Basic Redis
enabled     = Yes
config      = /etc/honeydb/redis.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `decoy_password` | string or null | `null` | Exact password that results in a fake authenticated session; null or empty disables the feature. |

#### INFO Events

| Event | Description |
|-------|-------------|
| `auth_attempt` | Captures the password (and ACL username on Redis 6+) supplied with an AUTH command. |
| `auth_result` | Fires when a submitted password matches the configured decoy password. |
| `config_set` | Captures a CONFIG SET key/value; tags config-based RCE setups. |
| `lua_script` | Captures an EVAL Lua script body; tags CVE-2022-0543 on sandbox-escape patterns. |
| `evalsha` | Captures the SHA passed to EVALSHA. |
| `slaveof` | Captures the host/port targeted by SLAVEOF/REPLICAOF. |
| `slaveof_no_one` | Fires on SLAVEOF/REPLICAOF NO ONE (replication detachment). |
| `module_load` | Captures a MODULE LOAD path used for module-injection RCE. |

### SIP_udp

A low interaction UDP SIP service. Emulates a SIP registrar/proxy to capture toll fraud attempts, credential stuffing (Digest auth brute-forcing via REGISTER floods), and reconnaissance probes from automated scanners. Known SIP scanner tools (sipvicious, friendly-scanner, sipsak, and others) are detected and flagged.

Example service configuration:

```shell
[SIP]
plugin      = SIP_udp
low_port    = udp:5060
port        = udp:5060
description = Session Initiation Protocol (SIP) for voice and video calls.
enabled     = Yes
config      = /etc/honeydb/sip.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `server_header` | string | `"Asterisk PBX"` | Value sent in the SIP `Server:` response header. Set to match the identity of a real SIP server (e.g. `"FreeSWITCH-mod_sofia/1.10.9"`, `"Cisco-SIPGateway/IOS-12.x"`). |

#### INFO Events

| Event | Description |
|-------|-------------|
| `sip_request` | Captures the method and key headers (User-Agent, From/To URI, Call-ID) of an incoming SIP request. |
| `options_probe` | Fires on a SIP OPTIONS request. |
| `sip_auth_attempt` | Captures Authorization/Proxy-Authorization credentials (scheme, username, realm, uri, algorithm). |
| `abuse_pattern` | Fires when the User-Agent matches a known scanner/abuse pattern. |

### SMTP_tcp

A low interaction TCP SMTP service. Attracts credential-stuffing attacks, open-relay probes, email enumeration attempts, and malware delivery activity. Captures sender/recipient addresses, authentication credentials, and email headers, and detects open-relay probes, header injection, and known-bad EHLO strings.

Example service configuration:

```shell
[SMTP]
plugin      = SMTP_tcp
low_port    = tcp:25
port        = tcp:25
description = SMTP service.
enabled     = Yes
config      = /etc/honeydb/smtp.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `server_name` | string | `"localhost"` | Hostname in the banner and EHLO response; also used as the local domain for open-relay detection. |
| `version_string` | string | `"ESMTP Exim 4.80"` | Version string appended to the banner. |

```
{
    "server_name": "mail.example.com",
    "version_string": "ESMTP Exim 4.96"
}
```

#### INFO Events

| Event | Description |
|-------|-------------|
| `helo_domain` | Captures the domain argument from EHLO/HELO. |
| `from_address` | Captures the sender from MAIL FROM. |
| `to_address` | Captures the recipient from RCPT TO. |
| `auth_type` | Records the requested AUTH mechanism (PLAIN/LOGIN). |
| `auth_credentials` | Captures the username/password from an AUTH PLAIN or AUTH LOGIN attempt. |
| `vrfy_address` | Captures the address probed via VRFY. |
| `expn_list` | Captures the mailing list probed via EXPN. |
| `email_header` | Captures selected DATA message headers (Subject, From, To, etc.). |
| `abuse_type` | Fires on an open-relay probe, header injection, or known-bad EHLO string. |

### SNMP_udp

A low interaction UDP SNMP service (SNMPv1/v2c). Responds to GET, GETNEXT, GETBULK, SET, TRAP, and INFORM requests with a simulated Cisco IOS router MIB. Captures community strings, OID requests, SET write attempts, and MIB walk enumeration, and detects known CVE probe patterns (CVE-2002-0013, CVE-2017-6736/6737/6738).

Example service configuration:

```shell
[SNMP]
plugin      = SNMP_udp
low_port    = udp:161
port        = udp:161
description = SNMP service.
enabled     = Yes
config      = /etc/honeydb/snmp.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `mib_overrides` | dict | `{}` | Map of OID string → string value to override default MIB values (e.g. set a custom `sysName` or `sysLocation`). |
| `community_string_length_threshold` | int | `64` | Byte length above which a community string triggers a CVE-2002-0013 probe event. |
| `mib_walk_max_repetitions_threshold` | int | `10` | GETBULK max-repetitions value above which MIB walk detection fires. |
| `mib_walk_getnext_count_threshold` | int | `5` | Cumulative GETNEXT count per source IP above which MIB walk detection fires. |

Example override to rename the device:

```
{
  "mib_overrides": {
    "1.3.6.1.2.1.1.5.0": "edge-rtr-01",
    "1.3.6.1.2.1.1.6.0": "Data Centre A"
  }
}
```

#### INFO Events

| Event | Description |
|-------|-------------|
| `snmp_request` | Captures each SNMP request (version, community, PDU type, OIDs). |
| `set_attempt` | Captures each varbind OID/value (and writability) on an SNMP SET. |
| `mib_walk_detected` | Fires on a suspected MIB walk (high GETBULK max-repetitions or repeated GETNEXT). |
| `cve_probe` | Fires on an oversized community string (CVE-2002-0013) or known Cisco CVE OID prefixes. |

### SSH_tcp

A medium interaction TCP SSH service based on Cowrie. To authenticate, use the user `root` with any password.

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

A low interaction UDP TFTP service. Serves a configurable decoy file for any read request and accepts write uploads (logging the first bytes without writing to disk). Detects abuse patterns including path traversal and IoT firmware download attempts.

Example service configuration:

```shell
[TFTP]
plugin      = TFTP_udp
low_port    = udp:69
port        = udp:69
description = TFTP service.
enabled     = Yes
config      = /etc/honeydb/tftp.json
```

#### Config

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `decoy_netascii` | string or null | `null` | Text content served for netascii read requests. Falls back to a built-in decoy file if absent. |
| `decoy_octet` | string or null | `null` | Base64-encoded bytes served for octet read requests. Falls back to a built-in decoy file if absent. |
| `upload_capture_bytes` | int | `512` | Maximum bytes captured from each upload for logging. |
| `max_upload_size_bytes` | int | `2000384` | Maximum total upload size before an error packet is sent. |
| `session_timeout_seconds` | int | `60` | Sessions with no activity for this many seconds are cleaned up. |
| `max_active_sessions` | int | `1000` | Cap on concurrent tracked sessions. |
| `iot_firmware_keywords` | list | `[]` | Additional filename keywords that trigger IoT firmware detection. Merged with the built-in list (`firmware`, `boot.img`, `uimage`, `bzimage`, `vmlinuz`, `rootfs`, `kernel.img`, `factory.bin`). |

#### INFO Events

| Event | Description |
|-------|-------------|
| `get_file` | Captures the filename and mode from an RRQ (read request). |
| `put_file` | Captures the filename and mode from a WRQ (write request). |
| `upload_data` | Captures buffered hex of an uploaded file (blocks received, completion/truncation flags). |
| `abuse_pattern` | Fires on path traversal, IoT firmware keywords, invalid transfer mode, or binary-in-netascii. |
| `parse_error` | Fires when a datagram cannot be parsed; captures the error and raw hex. |

### VNC_tcp

A low interaction TCP VNC service (RFB protocol). Implements the RFB handshake (versions 3.3/3.7/3.8), configurable security-type negotiation, dictionary-attack VNC authentication, and a synthetic post-auth desktop served as a static screenshot. Captures handshake details, authentication attempts, and post-auth keyboard/mouse/clipboard activity. Detects CVE-2006-2369 (RealVNC authentication bypass) probes.

Example service configuration:

```shell
[VNC]
plugin      = VNC_tcp
low_port    = tcp:5900
port        = tcp:5900
description = Low interaction VNC service
enabled     = Yes
config      = /etc/honeydb/vnc.json
```

#### Config

| Key | Default | Description |
|-----|---------|-------------|
| `rfb_version` | `"003.008"` | One of `"003.003"`, `"003.007"`, `"003.008"`. |
| `security_types_preset` | `"honeypot-default"` | One of `"tightvnc-like"`, `"tigervnc-like"`, `"realvnc-like"`, `"honeypot-default"`. |
| `security_types` | `null` | Explicit list of security type integers (overrides preset when set). |
| `server_name` | `"ubuntu-server"` | Desktop name sent to the client. |
| `framebuffer.width` / `framebuffer.height` | `1024` / `768` | Synthetic framebuffer dimensions. Screenshot images that don't match are excluded at load time. |
| `screenshots.directory` | `null` | Directory of same-dimension images. One is picked per source IP, so the same scanner sees the same "desktop" across reconnects. |
| `screenshots.path` | `null` | Single image (alternative to directory). |
| `screenshots.max_loaded` | `32` | Cap on cached screenshot images. |
| `password_wordlist_path` | `null` | Optional path to a password wordlist used for VNC-Auth; falls back to an embedded ~100-entry default. |
| `tcp_idle_timeout_seconds` | `60` | Per-connection idle timeout. |
| `max_message_bytes` | `8192` | Per-message size cap. |
| `max_buffer_bytes` | `65536` | Per-connection buffer cap. |
| `post_auth.framebuffer_enabled` | `true` | Switch for serving the screenshot framebuffer. |
| `post_auth.max_post_auth_messages_per_session` | `200` | Cap on post-auth client messages per session. |
| `structured_info_events` | `true` | `false` falls back to human-readable INFO strings. |
| `tag_known_abuse` | `true` | `false` suppresses abuse-pattern events. |

Security-type presets:

| Preset | Advertised types | Emulates |
|--------|-----------------|----------|
| `tightvnc-like` | `[16, 2]` | TightVNC server |
| `tigervnc-like` | `[2, 19]` | TigerVNC server |
| `realvnc-like` | `[2]` | RealVNC Free |
| `honeypot-default` | `[1, 2, 16, 19]` | Maximally attractive — all four probe surfaces |

!!! note
    Operator-supplied screenshot images are sent verbatim to attackers — never include real desktop content; use synthetic or generic images only.

#### INFO Events

| Event | Description |
|-------|-------------|
| `rfb_handshake` | Captures the server/client RFB versions and downgrade flag. |
| `security_choice` | Captures the advertised security types and the selected type. |
| `auth_attempt` | Captures the VNC-auth challenge/response and any matched wordlist password. |
| `client_init` | Captures the ClientInit shared-desktop flag. |
| `set_pixel_format` | Captures the client-supplied pixel format. |
| `set_encodings` | Captures the client's advertised encoding list and total count. |
| `framebuffer_update_request` | Captures the requested rectangle (x/y/w/h) and incremental flag. |
| `key_event` | Captures keyboard events (keysym, down flag, ASCII). |
| `pointer_event` | Captures pointer position and button mask. |
| `client_cut_text` | Captures clipboard cut-text length and content. |
| `abuse_pattern` | Fires on CVE/fingerprint probes (e.g. CVE-2006-2369) and malformed/oversize/unknown client messages. |

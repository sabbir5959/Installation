# Wazuh Custom Detection Lab Documentation

## Overview

This document describes custom Wazuh detections built in a lab using
Apache and a custom application log. The detection workflow is reusable
with Palo Alto, Cisco Secure Firewall/FTD, Fortinet, Nginx, IIS,
ModSecurity, or other products by adapting only the decoder/log source.

## Detection Flow

``` text
Log Source
   |
   v
Wazuh Agent
   |
   v
Decoder (Built-in or Custom)
   |
   v
Rule
   |
   v
Alert
   |
   v
Dashboard
```

## PHP Code Injection

-   Built-in decoder: web-accesslog
-   Rule ID: 100500
-   MITRE: T1190

Rule:

``` xml
<rule id="100500" level="12">
  <if_group>accesslog</if_group>
  <match>php://</match>
  <description>Possible PHP Code Injection Attempt</description>
  <group>web_attack,php_injection,</group>
  <mitre><id>T1190</id></mitre>
</rule>
```

Test:

``` bash
curl "http://localhost/index.php?page=php://input"
```

Future devices: - Palo Alto URL Logs - Cisco HTTP/Proxy Logs - Any
device exposing HTTP URL/URI

## HTML Injection

-   Built-in decoder: web-accesslog
-   Rule ID: 100620
-   MITRE: T1190

Rule:

``` xml
<rule id="100620" level="10">
  <if_group>accesslog</if_group>
  <match>Injected</match>
  <description>Possible HTML Injection Attempt</description>
  <group>web_attack,html_injection,</group>
  <mitre><id>T1190</id></mitre>
</rule>
```

Test:

``` bash
curl "http://localhost/index.php?name=<h1>Injected</h1>"
```

## XML Bomb

Reason for custom decoder: The application log was not recognized by a
built-in decoder ("No decoder matched").

Decoder:

``` xml
<decoder name="xml_bomb">
    <program_name>^myapp$</program_name>
    <prematch>ERROR XML entity expansion detected</prematch>
    <regex>^(ERROR XML entity expansion detected while parsing request.*)$</regex>
    <order>message</order>
</decoder>
```

Rule:

``` xml
<rule id="100610" level="10">
    <decoded_as>xml_bomb</decoded_as>
    <match>XML entity expansion detected</match>
    <description>Possible XML Bomb / XML Entity Expansion Attempt</description>
    <group>web_attack,xml_bomb,</group>
    <mitre><id>T1190</id></mitre>
</rule>
```

Test:

``` bash
logger -t myapp "ERROR XML entity expansion detected while parsing request"
```

## When to Create a Decoder

Create a custom decoder only when Phase 2 of `wazuh-logtest` shows:

    No decoder matched

If Phase 2 already identifies a decoder (for example `web-accesslog`,
`pam`, `sudo`, `sshd`), reuse the built-in decoder and create only a
custom rule.

## Migration Strategy (Apache -\> Palo Alto/Cisco)

1.  Identify the log source.
2.  Check whether Wazuh already has a decoder.
3.  If not, create a custom decoder.
4.  Reuse the same rule logic by matching the decoded fields (URL, URI,
    HTTP request, message, threat name, etc.).
5.  Validate with `wazuh-logtest`.
6.  Confirm alerts in Discover and dashboards.

## MITRE

  Detection            MITRE
  -------------------- -------
  PHP Code Injection   T1190
  HTML Injection       T1190
  XML Bomb             T1190

## Validation Checklist

-   Agent receives logs.
-   Decoder matches.
-   Rule matches.
-   Alert generated.
-   Dashboard shows the alert.
-   MITRE mapping verified.

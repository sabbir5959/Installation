# Suricata 8.x Installation and Wazuh Integration Guide

**Platform:** Ubuntu 24.04 LTS (also applicable to Ubuntu 22.04)**Integration:** Wazuh Agent → Wazuh Manager

## Overview

This guide covers:

1. Install Suricata
2. Configure network monitoring
3. Install Emerging Threats Open rules
4. Enable EVE JSON logging
5. Integrate Suricata with Wazuh Agent
6. Verify alerts on Wazuh Manager

---

## 1. Update System

```bash
sudo apt update && sudo apt upgrade -y
```

## 2. Install Dependencies

```bash
sudo apt install -y software-properties-common curl wget gnupg lsb-release ca-certificates jq
```

## 3. Add Official Repository

```bash
sudo add-apt-repository ppa:oisf/suricata-stable -y
sudo apt update
```

## 4. Install Suricata

```bash
sudo apt install -y suricata
```

## 5. Verify Installation

```bash
suricata --version
suricata --build-info
```

## 6. Update Rule Sources

```bash
sudo suricata-update update-sources
sudo suricata-update enable-source et/open
sudo suricata-update
sudo suricata-update list-enabled-sources
```

## 7. Verify Rules

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Expected:
- Configuration provided was successfully loaded.
- 52000+ rules successfully loaded

## 8. Identify Network Interface

```bash
ip -br addr
```

or

```bash
ip link show
```

Example:

- eth0 = Primary interface
- eth1 = Secondary interface

## 9. Backup Configuration

```bash
sudo cp /etc/suricata/suricata.yaml /etc/suricata/suricata.yaml.bak
```

## 10. Configure HOME_NET

```yaml
vars:
  address-groups:
    HOME_NET: "[192.168.121.0/24]"
    EXTERNAL_NET: "!$HOME_NET"
```

Replace the subnet if your environment uses a different network.

## 11. Configure AF_PACKET

```yaml
af-packet:
  - interface: eth0
    cluster-id: 99
    cluster-type: cluster_flow
    defrag: yes
```

Use the active monitoring interface.

## 12. Verify EVE JSON

Ensure:

```yaml
outputs:
  - eve-log:
      enabled: yes
      filetype: regular
      filename: eve.json
```

## 13. Enable Service

```bash
sudo systemctl enable --now suricata
sudo systemctl restart suricata
sudo systemctl status suricata
```

## 14. Verify Log Generation

```bash
sudo ls -lh /var/log/suricata
sudo tail -f /var/log/suricata/eve.json
```

## 15. Generate Test Traffic

```bash
curl http://testmynids.org/uid/index.html
```

## 16. Verify Events

HTTP events:

```bash
sudo grep '"event_type":"http"' /var/log/suricata/eve.json
```

Alert events:

```bash
sudo grep '"event_type":"alert"' /var/log/suricata/eve.json
```

## 17. Wazuh Integration

Backup:

```bash
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak
```

Edit:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

Restart:

```bash
sudo systemctl restart wazuh-agent
sudo systemctl status wazuh-agent
```

## 18. Verify on Wazuh Manager

```bash
sudo grep -i "suricata" /var/ossec/logs/alerts/alerts.json | tail -20
```

## Troubleshooting

### No rules loaded

```bash
sudo suricata-update
sudo systemctl restart suricata
```

### No alert events

```bash
sudo grep '"event_type":"alert"' /var/log/suricata/eve.json
```

### No events in Wazuh

Verify:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

Then restart:

```bash
sudo systemctl restart wazuh-agent
```

## Useful Commands

```bash
suricata --version
suricata --build-info
sudo suricata -T -c /etc/suricata/suricata.yaml
sudo systemctl status suricata
sudo systemctl restart suricata
sudo systemctl restart wazuh-agent
sudo tail -f /var/log/suricata/eve.json
```

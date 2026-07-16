# Suricata Installation and Wazuh Integration Guide

**Operating System:** Ubuntu 22.04 / Ubuntu 24.04 LTS

------------------------------------------------------------------------

## Overview

This guide covers:

1. Install Suricata
2. Configure network monitoring
3. Install Emerging Threats Open rules
4. Enable EVE JSON logging
5. Integrate Suricata with Wazuh Agent
6. Verify alerts on Wazuh Manager

---

## Step 1: Update the System

Update the package repository.

``` bash
sudo apt update
```

------------------------------------------------------------------------

## Step 2: Install Required Packages

Install required dependencies.

``` bash
sudo apt install -y software-properties-common curl wget gnupg lsb-release ca-certificates jq
```

------------------------------------------------------------------------

## Step 3: Add the Official Suricata Repository

``` bash
sudo add-apt-repository ppa:oisf/suricata-stable
```

Update the repository.

``` bash
sudo apt update
```

------------------------------------------------------------------------

## Step 4: Install Suricata

``` bash
sudo apt install suricata -y
```

------------------------------------------------------------------------

## Step 5: Verify Installation

Check the installed version.

``` bash
suricata --version
```

------------------------------------------------------------------------

## 6. Update Rule Sources

```bash
sudo suricata-update update-sources
sudo suricata-update enable-source et/open
sudo suricata-update
sudo suricata-update list-enabled-sources
```
------------------------------------------------------------------------

## 7. Verify Rules

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Expected:
- Configuration provided was successfully loaded.
- 52000+ rules successfully loaded

------------------------------------------------------------------------

## Step 7: Detect the Active Network Interface

Display all interfaces.

```bash
ip -br addr
```

or

```bash
ip link show
```

Example:

``` text
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
link/ether 00:11:22:33:44:55 brd ff:ff:ff:ff:ff:ff
inet 10.0.0.23/24 brd 10.23.0.255 scope global noprefixroute ens33
```

The active interface is:

``` text
ens33
```

------------------------------------------------------------------------

## Step 8: Backup the Suricata Configuration

``` bash
sudo cp /etc/suricata/suricata.yaml /etc/suricata/suricata.yaml.bak
```

------------------------------------------------------------------------

## Step 9: Edit the Suricata Configuration

``` bash
sudo nano /etc/suricata/suricata.yaml
```

------------------------------------------------------------------------

## Step 10: Configure HOME_NET

 ```yaml
 address-groups:
    HOME_NET: "[UBUNTU-IP]"
    #HOME_NET: "[192.168.0.0/16]"
    #HOME_NET: "[10.0.0.0/8]"
    #HOME_NET: "[172.16.0.0/12]"
    #HOME_NET: "any"

    EXTERNAL_NET: "!$HOME_NET"
    #EXTERNAL_NET: "any"
 ```
Replace the subnet if your environment uses a different network.

------------------------------------------------------------------------


## Step 11: Configure AF_PACKET

Locate the `af-packet:` section and configure it.

``` yaml
af-packet:
  - interface: ens33
    cluster-id: 99
    cluster-type: cluster_flow
    defrag: yes
```

Replace `ens33` with your active network interface.

If you want to use multiple interfaces, add one entry per interface.

``` yaml
af-packet:
  - interface: ens33
    cluster-id: 99
    cluster-type: cluster_flow
    defrag: yes

  - interface: ens34
    cluster-id: 100
    cluster-type: cluster_flow
    defrag: yes
```

Use a unique `cluster-id` for each interface entry.

------------------------------------------------------------------------

## Step 12: Enable EVE JSON Logging

Locate the `outputs:` section.

Ensure the following configuration exists.

``` yaml
outputs:

  - eve-log:
      enabled: yes
      filetype: regular
      filename: eve.json
```

------------------------------------------------------------------------

## 13. Enable Service

```bash
sudo systemctl enable --now suricata
sudo systemctl restart suricata
sudo systemctl status suricata
```

------------------------------------------------------------------------

## Step 13: Validate the Configuration

``` bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

------------------------------------------------------------------------

## Step 15: Edit the Suricata Rule Path

``` bash
sudo nano /etc/suricata/suricata.yaml
```

------------------------------------------------------------------------

## Step 16: Set rule-path direction

Locate the `rule-path:` section and edit it

 ```yaml
 default-rule-path: /etc/suricata/rules

 rule-files:
  - "*.rules"
 ```

<!-- By recommending this path, if you select it and open the rules directory there, any rules you create later will be saved inside it. This keeps all rule files together in one place. -->

------------------------------------------------------------------------

## Step 17: Restart the Suricata Service

``` bash
sudo systemctl restart suricata
sudo systemctl status suricata
```

------------------------------------------------------------------------

## Step 19: Verify Suricata Log Files

List the generated log files.

``` bash
sudo ls -lh /var/log/suricata
sudo tail -f /var/log/suricata/eve.json
```

------------------------------------------------------------------------

## 15. Generate Test Traffic

```bash
curl http://testmynids.org/uid/index.html
```

------------------------------------------------------------------------

## 17. Wazuh Integration

Backup:

```bash
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak
```
------------------------------------------------------------------------

## Step 21: Edit the Wazuh Agent Configuration

``` bash
sudo nano /var/ossec/etc/ossec.conf
```

------------------------------------------------------------------------

## Step 22: Add the Suricata Log Configuration

Add the following block inside the `<ossec_config>` section.

``` xml
<localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
</localfile>
```

------------------------------------------------------------------------

## Step 23: Restart the Wazuh Agent

``` bash
sudo systemctl restart wazuh-agent
```

------------------------------------------------------------------------

## Step 24: Verify the Wazuh Agent Status

``` bash
sudo systemctl status wazuh-agent
```

------------------------------------------------------------------------

## Step 25: Generate Test Traffic

Generate HTTP traffic.

``` bash
curl http://example.com
```

Generate HTTPS traffic.

``` bash
curl https://google.com
```

------------------------------------------------------------------------

## Step 26: Verify Suricata Events

Display the latest events.

``` bash
sudo tail -50 /var/log/suricata/eve.json
```

------------------------------------------------------------------------

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
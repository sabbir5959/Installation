# Suricata Installation and Wazuh Integration Guide

**Operating System:** Ubuntu 22.04 / Ubuntu 24.04 LTS

------------------------------------------------------------------------

## Step 1: Update the System

Update the package repository.

``` bash
sudo apt update
```

------------------------------------------------------------------------

## Step 2: Install Required Packages

Install required dependencies.

``` bash
sudo apt install software-properties-common jq curl -y
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

## Step 6: Update Detection Rules

Download the latest Emerging Threats rules.

``` bash
sudo suricata-update
```

------------------------------------------------------------------------

## Step 7: Detect the Active Network Interface

Display all interfaces.

``` bash
ip addr
```

or

``` bash
ip route
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

------------------------------------------------------------------------

## Step 11: Set rule-path direction

Locate the `rule-path:` section and edit it

 ```yaml
 default-rule-path: /etc/suricata/rules

 rule-files:
  - "*.rules"
 ```

------------------------------------------------------------------------

## Step 12: Configure AF_PACKET

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

## Step 13: Enable EVE JSON Logging

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

## Step 14: Validate the Configuration

``` bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

------------------------------------------------------------------------

## Step 15: Enable the Suricata Service

``` bash
sudo systemctl enable suricata
```

------------------------------------------------------------------------

## Step 16: Restart the Suricata Service

``` bash
sudo systemctl restart suricata
```

------------------------------------------------------------------------

## Step 17: Check the Suricata Service Status

``` bash
sudo systemctl status suricata
```

------------------------------------------------------------------------

## Step 18: Verify Suricata Log Files

List the generated log files.

``` bash
ls -lah /var/log/suricata/
```

Monitor events in real time.

``` bash
sudo tail -f /var/log/suricata/eve.json
```

------------------------------------------------------------------------

## Step 19: Backup the Wazuh Agent Configuration

``` bash
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak
```

------------------------------------------------------------------------

## Step 20: Edit the Wazuh Agent Configuration

``` bash
sudo nano /var/ossec/etc/ossec.conf
```

------------------------------------------------------------------------

## Step 21: Add the Suricata Log Configuration

Add the following block inside the `<ossec_config>` section.

``` xml
<localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
</localfile>
```

------------------------------------------------------------------------

## Step 22: Restart the Wazuh Agent

``` bash
sudo systemctl restart wazuh-agent
```

------------------------------------------------------------------------

## Step 23: Verify the Wazuh Agent Status

``` bash
sudo systemctl status wazuh-agent
```

------------------------------------------------------------------------

## Step 24: Generate Test Traffic

Generate HTTP traffic.

``` bash
curl http://example.com
```

Generate HTTPS traffic.

``` bash
curl https://google.com
```

------------------------------------------------------------------------

## Step 25: Verify Suricata Events

Display the latest events.

``` bash
sudo tail -50 /var/log/suricata/eve.json
```

------------------------------------------------------------------------

## Step 26: Useful Commands

Check the installed version.

``` bash
suricata --version
```

Display build information.

``` bash
suricata --build-info
```

Restart the Suricata service.

``` bash
sudo systemctl restart suricata
```

Check the Suricata service.

``` bash
sudo systemctl status suricata
```

Restart the Wazuh Agent.

``` bash
sudo systemctl restart wazuh-agent
```

Check the Wazuh Agent.

``` bash
sudo systemctl status wazuh-agent
```

Monitor the Suricata log.

``` bash
sudo tail -f /var/log/suricata/eve.json
```

Validate the configuration.

``` bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

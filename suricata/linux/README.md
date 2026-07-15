# Suricata Installation and Wazuh Integration Guide

**Operating System:** Ubuntu 22.04 / Ubuntu 24.04 LTS

------------------------------------------------------------------------

## Step 1: Update the System

Update the package repository.

``` bash
sudo apt update
```

Upgrade installed packages.

``` bash
sudo apt upgrade -y
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

or

``` bash
suricata --build-info
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
default via 192.168.1.1 dev ens33
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

## Step 10: Configure AF_PACKET

Locate the `af-packet:` section and configure it.

``` yaml
af-packet:
  - interface: ens33
    cluster-id: 99
    cluster-type: cluster_flow
    defrag: yes
```

Replace `ens33` with your active network interface.

------------------------------------------------------------------------

## Step 11: Enable EVE JSON Logging

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

## Step 12: Validate the Configuration

``` bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

------------------------------------------------------------------------

## Step 13: Enable the Suricata Service

``` bash
sudo systemctl enable suricata
```

------------------------------------------------------------------------

## Step 14: Restart the Suricata Service

``` bash
sudo systemctl restart suricata
```

------------------------------------------------------------------------

## Step 15: Check the Suricata Service Status

``` bash
sudo systemctl status suricata
```

------------------------------------------------------------------------

## Step 16: Verify Suricata Log Files

List the generated log files.

``` bash
ls -lah /var/log/suricata/
```

Monitor events in real time.

``` bash
sudo tail -f /var/log/suricata/eve.json
```

------------------------------------------------------------------------

## Step 17: Backup the Wazuh Agent Configuration

``` bash
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak
```

------------------------------------------------------------------------

## Step 18: Edit the Wazuh Agent Configuration

``` bash
sudo nano /var/ossec/etc/ossec.conf
```

------------------------------------------------------------------------

## Step 19: Add the Suricata Log Configuration

Add the following block inside the `<ossec_config>` section.

``` xml
<localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
</localfile>
```

------------------------------------------------------------------------

## Step 20: Restart the Wazuh Agent

``` bash
sudo systemctl restart wazuh-agent
```

------------------------------------------------------------------------

## Step 21: Verify the Wazuh Agent Status

``` bash
sudo systemctl status wazuh-agent
```

------------------------------------------------------------------------

## Step 22: Generate Test Traffic

Generate HTTP traffic.

``` bash
curl http://example.com
```

Generate HTTPS traffic.

``` bash
curl https://google.com
```

------------------------------------------------------------------------

## Step 23: Verify Suricata Events

Display the latest events.

``` bash
sudo tail -50 /var/log/suricata/eve.json
```

------------------------------------------------------------------------

## Step 24: Useful Commands

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

# psono-wazuh-integration
Forward Psono password manager's audit logs into wazuh with proper decoders and rules.

# Setup Process
This guide assumes a familiarity with docker-compose and Wazuh. You'll also need to have your Wazuh manager and agent setup in addition to an existing Psono install. While this setup is intended for running multiple Psono instances on the same host, you can use it with just one.

## The Log Files
We will be storing the Psono logs in `/var/logs/psono/`. You will have a `.log` file for each Psono container. 
Each `.log` file will be mounted in its docker container with a bind mount:
```
services:
  psono:
    volumes:
      - type: bind
        source: /var/log/psono/psono-server-1.log
        target: /var/log/psono/audit.log
```
The file must be created before starting the container: `touch /var/log/psono/psono-server-1.log`

## Other Container Configuration
Psono Audit logs are only available in the enterprise version of the container:
```
services:
  psono:
    image: psono/psono-combo-enterprise:latest
```
The combo image is used here, but the regular server image also works: `psono/psono-server-enterprise:latest`

We also need to set a hostname in the container so Wazuh can properly identifiy the seperate Psono containers:
```
services:
  psono:
    hostname: psono-server-1
```

## Wazuh Agent Configuration
We need to add the `/var/log/psono/` directory to the wazuh-agent's config file `/var/ossec/etc/ossec.conf`
```
<ossec_config>
  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/psono/*.log</location>
    <out_format>psono_audit $(log)</out_format>
  </localfile>
</ossec_config>
```
This configuration can also be used on a wazuh-manager install.

## Wazuh Ruleset
Add the following files to your wazuh manager install.
- `psono-decoders.xml` --> `/var/ossec/etc/decoders/`
- `psono-rules.xml` --> `/var/ossec/etc/rules/` **The psono-rules.xml is in a primitive state, only basic logging of all events.**

Alternativly you can add them through the web interface.

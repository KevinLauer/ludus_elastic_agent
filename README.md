# Ansible Role: Elastic Agent Deployment

An Ansible role that deploys Elastic Agents to Windows, Debian, and Ubuntu systems.

## Description

- The role checks if the Elastic Agents have been downloaded to the Ludus host. If not, it will attempt to download the agents based on the `ludus_elastic_agent_version` variable.
- Agent versions can be [found here](https://www.elastic.co/downloads/past-releases#elastic-agent)
- The role is designed to work with Windows, Debian, Ubuntu systems.
- This fork complements [KevinLauer/ludus_elastic_container](https://github.com/KevinLauer/ludus_elastic_container) (detect and prevent agent policies). Upstream is [badsectorlabs/ludus_elastic_agent](https://github.com/badsectorlabs/ludus_elastic_agent).

Warning:

- `--force` flag is used during agent installation. This overwrites the current installation and does not prompt for confirmation.
- `--insecure` flag is used during agent installation. This is to ignore the self-signed certs.

## Requirements

None.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    # The ludus_elastic_container role will output this to the console if you're monitoring the logs.
    # Also accessible via the kibana UI.
    # Also accessible in {{ ludus_elastic_container_install_path }}/enrollment_token_<detect|prevent>.txt
    ludus_elastic_enrollment_token: ""

    # the IP address of your elastic server and port (defaults to 8220)
    # `ludus range status` will provide you with the IP address
    ludus_elastic_fleet_server: ""

    # A valid agent version to download and install
    ludus_elastic_agent_version: "9.4.2"

    # Install Sysmon on any Windows host (Elastic v9.X ingests the log)
    ludus_elastic_install_sysmon: ""

    # Sysmon install location
    ludus_elastic_sysmon_path: "C:\\Program Files (x86)\\Sysmon"

    # Agent mode: "detect" or "prevent". Selects the matching Fleet enrollment token.
    ludus_elastic_agent_mode: "detect"

## Dependencies

None.

## Example Playbook

```yaml
- hosts: elastic-agent
  roles:
    - ludus_elastic_agent
  role_vars:
    ludus_elastic_enrollment_token: "<TOKEN>"
    ludus_elastic_fleet_server: "https://<IP>:8220" # :8220 by default
    ludus_elastic_agent_version: "9.4.2"
    ludus_elastic_agent_mode: "detect" # or "prevent"
```

## Example Ludus Range Config

```yaml
ludus:
  - vm_name: "{{ range_id }}-win11-detect"
    hostname: "{{ range_id }}-WIN11-detect"
    template: win11-25h2-x64-enterprise-template
    vlan: 10
    ip_last_octet: 21
    ram_gb: 8
    cpus: 4
    windows:
      install_additional_tools: false
    roles:
      - name: ludus_elastic_agent
        depends_on:
          - vm_name: "{{ range_id }}-elastic-server"
            role: ludus_elastic_container
    role_vars:
      ludus_elastic_agent_mode: "detect"

  - vm_name: "{{ range_id }}-win11-prevent"
    hostname: "{{ range_id }}-WIN11-prevent"
    template: win11-25h2-x64-enterprise-template
    vlan: 10
    ip_last_octet: 22
    ram_gb: 8
    cpus: 4
    windows:
      install_additional_tools: false
    roles:
      - name: ludus_elastic_agent
        depends_on:
          - vm_name: "{{ range_id }}-elastic-server"
            role: ludus_elastic_container
    role_vars:
      ludus_elastic_agent_mode: "prevent"
```

Set the `role_vars` to install Elastic agent v8.X:

```yaml
ludus:
  - vm_name: "{{ range_id }}-jumpbox01"
    hostname: "{{ range_id }}-jumpbox01"
    template: win2022-server-x64-template
    vlan: 20
    ip_last_octet: 21
    ram_gb: 6
    cpus: 4
    windows:
      sysprep: true
    roles:
      - ludus_elastic_agent
    role_vars:
      ludus_elastic_agent_version: "8.12.2"
      ludus_elastic_install_sysmon: false
```

## Ludus setup

This role must be installed manually on the Ludus host (it is not on Ansible Galaxy). Use the path to the directory that contains this role:

```bash
git clone https://github.com/KevinLauer/ludus_elastic_agent.git
ludus ansible role add -d ./ludus_elastic_agent
```

Then:

```
# Get your config into a file so you can assign to your VMs
ludus range config get > config.yml

# Edit config to add the role to the VMs you wish to make an elastic server
ludus range config set -f config.yml

# Deploy the range with the user-defined-roles ONLY :)
ludus range deploy -t user-defined-roles
```

## License

GPLv3

## Author Information

This role was created by [Bad Sector Labs](https://badsectorlabs.com/), for [Ludus](https://ludus.cloud/). This fork is maintained at [KevinLauer/ludus_elastic_agent](https://github.com/KevinLauer/ludus_elastic_agent).

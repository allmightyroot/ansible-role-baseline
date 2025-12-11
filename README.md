# Ansible Role: baseline

[![CI](https://github.com/allmightyroot/ansible-role-baseline/workflows/CI/badge.svg)](https://github.com/allmightyroot/ansible-role-baseline/actions)

## Description

Establishes baseline system configuration for cloud instances. This role performs initial system setup including package updates, installation of common utilities, cloud provider detection, and optional tooling for AWS, SSH key rotation, and system change tracking.

**Key responsibilities:**
- System package updates (OS family-aware)
- Installation of base packages (curl, git, vim, tmux, etc.)
- Cloud provider detection (AWS, Azure, GCP, or none)
- Conditional AWS tooling installation (CLI, SSM Agent, CloudWatch Agent)
- ZSH installation and configuration
- SSH host key rotation via systemd service
- System change tracking via etckeeper

## Requirements

- Ansible >= 2.9
- `community.general` collection >= 5.0.0

## Role Variables

All variables are prefixed with `baseline_` and can be found in `defaults/main.yml`:

### Cloud Provider Detection
- `baseline_detect_cloud_provider: true` - Enable automatic cloud provider detection

### System Updates
- `baseline_update_system: true` - Update all system packages on first run
- `baseline_update_cache_valid_time: 3600` - Cache validity for apt (in seconds)

### Base Packages
- `baseline_packages` - List of packages to install on all systems

### Optional Components
- `baseline_install_zsh: true` - Install and configure ZSH
- `baseline_zsh_default_shell: false` - Set ZSH as default shell (requires `baseline_install_zsh: true`)

### AWS Tooling (only installs on AWS)
- `baseline_install_aws_cli: true` - Install AWS CLI
- `baseline_aws_cli_version: "2"` - AWS CLI version ("1" or "2")
- `baseline_install_ssm_agent: true` - Install AWS Systems Manager agent
- `baseline_ssm_agent_enabled: true` - Enable SSM agent service
- `baseline_install_cloudwatch_agent: true` - Install CloudWatch agent
- `baseline_cloudwatch_agent_enabled: false` - Enable CloudWatch agent (requires configuration)

### System Tools
- `baseline_install_etckeeper: true` - Install etckeeper for `/etc` change tracking
- `baseline_etckeeper_vcs: "git"` - Version control for etckeeper ("git" or "hg")
- `baseline_rotate_ssh_keys: true` - Enable SSH host key rotation on first boot

## Cloud Provider Detection

The role automatically detects the cloud provider by querying metadata services:

- **AWS**: http://169.254.169.254/latest/meta-data/
- **Azure**: http://169.254.169.254/metadata/instance (with Metadata: true header)
- **GCP**: http://metadata.google.internal/computeMetadata/v1/ (with Metadata-Flavor: Google header)
- **None**: If no metadata service responds, sets `cloud_provider: "none"`

This detection happens in `tasks/detect-cloud-provider.yml` and sets the `cloud_provider` fact for use by conditional tasks.

## OS Support

The role supports multiple Linux distributions through OS family detection:

- **Debian family**: Ubuntu, Debian (uses apt)
- **RedHat family**: CentOS, RHEL, Rocky, AlmaLinux (uses yum/dnf)
- **SUSE family**: SLES, openSUSE (uses zypper)

Update tasks must exist for each family being used.

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Configure baseline system
  hosts: all
  become: yes

  roles:
    - role: baseline
      vars:
        baseline_update_system: true
        baseline_install_zsh: true
        baseline_install_aws_cli: true
        baseline_install_ssm_agent: true
        baseline_rotate_ssh_keys: true
```

## Testing

To test this role locally:

```bash
ansible-playbook tests/test.yml -i tests/inventory
```

To test with Molecule (if configured):

```bash
molecule test
```

## License

MIT-0

## Author Information

Created by allmightyroot

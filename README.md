# HCV Deploy Files

[![License](https://img.shields.io/badge/license-copyleft-blue.svg)](LICENSE)
[![Ansible Role](https://img.shields.io/badge/ansible--galaxy-ns.hcvdeployfiles-blue.svg)](https://galaxy.ansible.com/ns/hcvdeployfiles)
[![Molecule Test](https://github.com/truestory1/hcv-deploy-files/actions/workflows/molecule.yml/badge.svg)](https://github.com/truestory1/hcv-deploy-files/actions/workflows/molecule.yml)

An Ansible role for deploying configuration files across servers. This role is part of the HCV (Host Configuration Versioning) system that automates the deployment and management of server configurations.

## Table of Contents

- [Overview](#overview)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Features](#features)
- [Usage](#usage)
  - [Basic Playbook Example](#basic-playbook-example)
  - [Advanced Configuration](#advanced-configuration)
  - [Configuration Examples](#configuration-examples)
  - [Task Overview](#task-overview)
- [Integration with HCV System](#integration-with-hcv-system)
- [Project Structure](#project-structure)
- [Testing](#testing)
- [Development](#development)
- [Contributing](#contributing)
- [Troubleshooting](#troubleshooting)
- [Dependencies](#dependencies)
- [License](#license)
- [Support](#support)

## Overview

This Ansible role provides a standardized approach to deploy configuration files from various repositories to target servers. It handles file permissions, ownership, and directory structures according to predefined configuration specifications.

## Requirements

### System Requirements
- **Ansible**: >= 2.10
- **Python**: >= 3.6
- **Target OS**: Rocky Linux 9, RHEL 9, or compatible EL 9 distributions
- **Git**: Required on target systems for repository operations

### Control Node Requirements
- Ansible installed and configured
- Network access to target servers
- SSH access to target servers with appropriate privileges

### Target Server Requirements
- Git package installed
- Tree package installed (for display output)
- Sudo privileges for the Ansible user (if modifying system files)

## Installation

### From Ansible Galaxy

```bash
# Install the role from Ansible Galaxy
ansible-galaxy install ns.hcvdeployfiles
```

### From Source

```bash
# Clone the repository
git clone https://github.com/truestory1/hcv-deploy-files.git
cd hcv-deploy-files

# Install development dependencies (optional, for testing)
pip install molecule molecule-docker ansible-lint yamllint
```

### Dependencies Installation

Install required packages on target servers:
```bash
# On Rocky Linux/RHEL 9
sudo dnf install git tree
```

## Quick Start

1. **Install the role:**
   ```bash
   ansible-galaxy install ns.hcvdeployfiles
   ```

2. **Create a basic playbook:**
   ```bash
   cat > deploy-configs.yml << 'EOF'
   ---
   - name: Deploy server configurations
     hosts: all
     become: true
     roles:
       - role: ns.hcvdeployfiles
   EOF
   ```

3. **Set up your inventory with HCV configuration:**
   ```bash
   cat > inventory.yml << 'EOF'
   all:
     hosts:
       server01:
         ansible_host: 192.168.1.100
         git_repo: "https://github.com/yourorg/hcv-servers"
         git_version: "main"
   EOF
   ```

4. **Run the playbook:**
   ```bash
   ansible-playbook -i inventory.yml deploy-configs.yml
   ```

## Project Structure

```
hcv-deploy-files/
├── README.md              # This documentation
├── renovate.json          # Dependency management
├── defaults/
│   └── main.yml          # Default variables
├── handlers/
│   └── main.yml          # Event handlers
├── meta/
│   └── main.yml          # Role metadata
├── molecule/
│   └── default/          # Testing framework
│       ├── converge.yml
│       ├── Dockerfile.j2
│       ├── molecule.yml
│       ├── prepare.yml
│       └── verify.yml
├── tasks/
│   ├── config.yml        # Configuration tasks
│   ├── main.yml          # Main task entry point
│   └── server.yml        # Server-specific tasks
└── vars/
    └── main.yml          # Role variables
```

## Features

- **Automated Configuration Deployment**: Deploy files and directories with proper permissions
- **Multi-Server Support**: Handle different server configurations simultaneously
- **Permission Management**: Set file/directory ownership and permissions
- **Git Integration**: Support for git-based configuration repositories
- **Testing Framework**: Molecule-based testing for reliability
- **Idempotent Operations**: Safe to run multiple times

## Usage

### Basic Playbook Example

```yaml
---
- name: Deploy configurations
  hosts: all
  become: true
  roles:
    - role: ns.hcvdeployfiles
```

### Advanced Configuration

The role works with configuration repositories that follow the HCV format. The role expects specific variables to be defined:

```yaml
# In your inventory or group_vars
git_repo: "https://github.com/yourorg/hcv-servers"
git_version: "main"  # or any git branch/tag/commit
```

#### Server Configuration Example

In your server inventory (e.g., `hcv-servers/server01.yaml`):
```yaml
---
- name: Config01
  version: master
  repo: https://github.com/yourorg/hcv-config-01
```

#### Configuration Repository Structure

Each configuration repository should contain:
- **Files to deploy**: Organized in the desired directory structure
- **`.config.yml`**: Defines deployment rules, permissions, and ownership

Example `.config.yml`:
```yaml
---
directories:
  - name: "etc/nginx"
    mode: "0755"
    owner: "root"
    group: "root"
  - name: "var/log/myapp"
    mode: "0755"
    owner: "myapp"
    group: "myapp"

files:
  - file: "etc/nginx/nginx.conf"
    mode: "0644"
    owner: "root"
    group: "root"
  - file: "etc/systemd/system/myapp.service"
    mode: "0644"
    owner: "root"
    group: "root"
```

### Configuration Examples

#### Multiple Configuration Repositories

```yaml
# In hcv-servers/server01.yaml
---
- name: WebServer Config
  version: v1.2.0
  repo: https://github.com/yourorg/hcv-webserver-config
- name: Database Config  
  version: main
  repo: https://github.com/yourorg/hcv-database-config
- name: Monitoring Config
  version: latest
  repo: https://github.com/yourorg/hcv-monitoring-config
```

#### Environment-Specific Deployments

```yaml
# Inventory for production
production:
  hosts:
    prod-web-01:
      git_repo: "https://github.com/yourorg/hcv-servers"
      git_version: "production"
    prod-web-02:
      git_repo: "https://github.com/yourorg/hcv-servers"  
      git_version: "production"

# Inventory for staging
staging:
  hosts:
    stage-web-01:
      git_repo: "https://github.com/yourorg/hcv-servers"
      git_version: "staging"
```

### Task Overview

1. **main.yml**: Entry point that displays tree output and includes server tasks
2. **server.yml**: Handles server-specific configuration deployment
3. **config.yml**: Manages individual configuration file deployments

## Testing

This role includes comprehensive testing using Molecule:

### Prerequisites
- Docker or Podman
- Molecule
- Ansible

### Running Tests

```bash
# Install molecule
pip install molecule molecule-docker

# Run all tests
molecule test

# Run specific scenarios
molecule converge
molecule verify
```

### Test Environment

The testing setup includes:
- **prepare.yml**: Installs dependencies (git, tree) and configures git safe directories
- **converge.yml**: Applies the role to test instances
- **verify.yml**: Validates the deployment results
- **Dockerfile.j2**: Defines the test container environment

## Development

### Local Development Setup

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd hcv-deploy-files
   ```

2. Install development dependencies:
   ```bash
   pip install molecule molecule-docker ansible-lint
   ```

3. Make your changes and test:
   ```bash
   molecule test
   ```

### Adding New Features

1. Update the appropriate task files in `tasks/`
2. Add or modify variables in `defaults/main.yml` or `vars/main.yml`
3. Update handlers in `handlers/main.yml` if needed
4. Add tests in the `molecule/` directory
5. Update this README with new features

## Integration with HCV System

The HCV (Host Configuration Versioning) system provides a comprehensive approach to managing server configurations using Git repositories and Ansible automation.

### Repository Architecture

This role is designed to work seamlessly with:

- **[hcv-config-01](https://github.com/truestory1/hcv-config-01)**: Configuration repository defining file permissions and structure
- **[hcv-servers](https://github.com/truestory1/hcv-servers)**: Server inventory and configuration specifications
- **Additional hcv-config-* repositories**: For different service configurations

### Workflow Overview

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│  hcv-servers    │    │  hcv-config-*    │    │  Target Server  │
│  Repository     │    │  Repositories    │    │                 │
│                 │    │                  │    │                 │
│ server01.yaml ──┼────▶ Configuration   │    │                 │
│ server02.yaml   │    │ Files +          │    │                 │
│ ...             │    │ .config.yml      │    │                 │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       ▲
         │                       │                       │
         │              ┌────────▼────────┐             │
         │              │                 │             │
         └──────────────▶│  hcv-deploy-   │─────────────┘
                         │  files (this   │
                         │  Ansible role) │
                         │                 │
                         └─────────────────┘
```

### Detailed Workflow Steps

1. **Configuration Management**:
   - Configuration files are stored in `hcv-config-*` repositories
   - Each repository contains files and a `.config.yml` with deployment rules
   - Changes are version-controlled and tagged for releases

2. **Server Specification**:
   - Server configurations are defined in `hcv-servers` repository
   - Each server has a YAML file listing required configurations
   - Versions can be pinned to specific releases or branches

3. **Deployment Process**:
   - This role reads server specifications from `hcv-servers`
   - Downloads each referenced configuration repository
   - Applies files and permissions according to `.config.yml`
   - Ensures idempotent operations

4. **Permission Management**:
   - File permissions and ownership are defined in `.config.yml`
   - Directories are created with proper permissions before file deployment
   - Support for different users, groups, and permission modes

### Example Integration

**Step 1**: Create a configuration repository (`hcv-config-webserver`)
```bash
mkdir hcv-config-webserver
cd hcv-config-webserver

# Create configuration files
mkdir -p etc/nginx/conf.d
echo "server { ... }" > etc/nginx/nginx.conf
echo "server { ... }" > etc/nginx/conf.d/default.conf

# Create deployment rules
cat > .config.yml << 'EOF'
---
directories:
  - name: "etc/nginx"
    mode: "0755"
    owner: "root"
    group: "root"
  - name: "etc/nginx/conf.d"
    mode: "0755"
    owner: "root"
    group: "root"

files:
  - file: "etc/nginx/nginx.conf"
    mode: "0644"
    owner: "root"
    group: "root"
  - file: "etc/nginx/conf.d/default.conf"
    mode: "0644"
    owner: "root"
    group: "root"
EOF

git init
git add .
git commit -m "Initial webserver configuration"
git tag v1.0.0
```

**Step 2**: Define server configuration (`hcv-servers/webserver01.yaml`)
```yaml
---
- name: WebServer Config
  version: v1.0.0
  repo: https://github.com/yourorg/hcv-config-webserver
```

**Step 3**: Deploy using this role
```bash
ansible-playbook -i inventory.yml deploy-configs.yml
```

## Dependencies

### Ansible Dependencies
This role has no external Ansible role dependencies, making it lightweight and easy to integrate.

### System Dependencies
- **git**: Required for repository operations
- **tree**: Used for directory structure display (optional)
- **sudo**: Required if deploying files requiring elevated privileges

### Python Dependencies
- **ansible**: >= 2.10
- **GitPython**: Installed automatically with Ansible

### Compatibility Matrix

| Component | Version | Status |
|-----------|---------|---------|
| Ansible | 2.10+ | ✅ Supported |
| Ansible | 2.9 | ⚠️ May work |
| Rocky Linux | 9 | ✅ Tested |
| RHEL | 9 | ✅ Supported |
| CentOS Stream | 9 | ✅ Should work |
| Ubuntu | 20.04+ | ⚠️ Untested |
| Python | 3.6+ | ✅ Required |

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Add or update tests
5. Run tests (`molecule test`)
6. Commit your changes (`git commit -m 'Add amazing feature'`)
7. Push to the branch (`git push origin feature/amazing-feature`)
8. Open a Pull Request

## Troubleshooting

### Common Issues

#### Git Safe Directory Warnings
**Problem**: Git warnings about safe directories in test environments
```
fatal: detected dubious ownership in repository
```
**Solution**: The role automatically configures git safe directories in the test environment. For manual setup:
```bash
git config --global --add safe.directory '*'
```

#### Permission Errors
**Problem**: Permission denied when creating files or directories
```
TASK [ns.hcvdeployfiles : Create required directories] *** FATAL: [server01]: FAILED! => {"msg": "Permission denied"}
```
**Solutions**:
- Ensure the Ansible user has appropriate sudo privileges
- Add `become: true` to your playbook
- Check file ownership and permissions in `.config.yml`

#### Missing Dependencies
**Problem**: Required packages not found on target systems
```
TASK [ns.hcvdeployfiles : Clone git repository] *** FATAL: [server01]: FAILED! => {"msg": "git command not found"}
```
**Solution**: The prepare phase installs required packages. For manual installation:
```bash
# Rocky Linux/RHEL 9
sudo dnf install git tree

# Ubuntu/Debian
sudo apt-get install git tree
```

#### Repository Access Issues
**Problem**: Cannot clone configuration repositories
```
TASK [ns.hcvdeployfiles : Clone git repository] *** FATAL: [server01]: FAILED! => {"msg": "Repository not found"}
```
**Solutions**:
- Verify repository URL is correct and accessible
- Check network connectivity from target servers
- Ensure proper authentication for private repositories
- Verify the git version/branch exists

#### Configuration File Not Found
**Problem**: Server configuration file missing
```
TASK [ns.hcvdeployfiles : Use config file from git as variable] *** FATAL: [server01]: FAILED! => {"msg": "file not found: server01.yaml"}
```
**Solutions**:
- Ensure server hostname matches the YAML filename in hcv-servers repository
- Check that the file exists in the correct repository branch
- Verify the `git_repo` and `git_version` variables are set correctly

#### Invalid Configuration Format
**Problem**: Configuration file parsing errors
```
TASK [ns.hcvdeployfiles : Set variable from config file] *** FATAL: [server01]: FAILED! => {"msg": "YAML parse error"}
```
**Solutions**:
- Validate YAML syntax in configuration files
- Check that `.config.yml` follows the expected format
- Use `yamllint` to check file syntax

### Debug Mode

Run with increased verbosity for troubleshooting:
```bash
# Ansible playbook with maximum verbosity
ansible-playbook -vvv your-playbook.yml

# Molecule testing with debug
molecule --debug test

# Check specific tasks
ansible-playbook --start-at-task="Clone git repository" -vvv your-playbook.yml
```

### Validation Commands

Verify your setup before deployment:

```bash
# Test git repository access
git clone <your-hcv-servers-repo> /tmp/test-clone
ls /tmp/test-clone/

# Validate YAML files
yamllint hcv-servers/server01.yaml
yamllint hcv-config-01/.config.yml

# Test Ansible connectivity
ansible all -i inventory.yml -m ping

# Check sudo access
ansible all -i inventory.yml -m command -a "sudo whoami" --become
```

### Log Analysis

Check logs for detailed error information:

```bash
# System logs
sudo journalctl -f

# Ansible logs (if using callback plugins)
cat ~/.ansible.log

# Check file permissions
ls -la /path/to/deployed/files
```

## License

This project is licensed under copyleft terms.

## Support

### Getting Help

For issues and questions:

1. **Documentation**: Check this README and inline code comments
2. **GitHub Issues**: [Create a new issue](https://github.com/truestory1/hcv-deploy-files/issues/new) with:
   - Description of the problem
   - Steps to reproduce
   - Expected vs actual behavior
   - Ansible version and target OS
   - Relevant log output (use `-vvv` for verbose logs)
3. **Examples**: Review the molecule test scenarios in `molecule/` directory
4. **Community**: Check existing GitHub issues for similar problems

### Reporting Bugs

When reporting bugs, please include:

- **Environment details**:
  - Ansible version (`ansible --version`)
  - Target OS and version
  - Python version
- **Configuration**:
  - Playbook code (sanitized)
  - Inventory configuration
  - Variable definitions
- **Error output**:
  - Full error messages
  - Debug output (`ansible-playbook -vvv`)
  - Relevant log files

### Feature Requests

For new features:
1. Check existing issues for similar requests
2. Describe the use case and expected behavior
3. Provide examples of how the feature would be used
4. Consider contributing a pull request

### Contributing Documentation

Help improve this documentation:
- Fix typos or unclear sections
- Add more examples
- Improve troubleshooting guides
- Translate to other languages

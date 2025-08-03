# HCV Deploy Files

An Ansible role for deploying configuration files across servers. This role is part of the HCV (Host Configuration Versioning) system that automates the deployment and management of server configurations.

## Overview

This Ansible role provides a standardized approach to deploy configuration files from various repositories to target servers. It handles file permissions, ownership, and directory structures according to predefined configuration specifications.

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
  roles:
    - role: ns.hcvdeployfiles
```

### Advanced Configuration

The role works with configuration repositories that follow the HCV format:

```yaml
# In your server inventory (e.g., hcv-servers/server01.yaml)
---
- name: Config01
  version: master
  repo: /config/hcv-config-01
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

This role is designed to work seamlessly with:

- **[hcv-config-01](https://github.com/truestory1/hcv-config-01)**: Configuration repository defining file permissions and structure
- **[hcv-servers](https://github.com/truestory1/hcv-servers)**: Server inventory and configuration specifications

### Workflow

1. Configuration changes are made in `hcv-config-*` repositories
2. Server specifications reference these configurations in `hcv-servers`
3. This role deploys the configurations to target servers
4. File permissions and ownership are applied according to `.config.yml` specifications

## Dependencies

This role has no external Ansible role dependencies, making it lightweight and easy to integrate.

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

- **Git safe directory warnings**: The role automatically configures git safe directories in the test environment
- **Permission errors**: Ensure the Ansible user has appropriate sudo privileges
- **Missing dependencies**: The prepare phase installs required packages (git, tree)

### Debug Mode

Run with increased verbosity for troubleshooting:
```bash
ansible-playbook -vvv your-playbook.yml
```

## License

This project is licensed under copyleft terms.

## Support

For issues and questions:
1. Check the existing GitHub issues
2. Review the molecule test scenarios for examples
3. Create a new issue with detailed information about your problem

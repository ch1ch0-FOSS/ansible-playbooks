# ansible-playbooks

Production Ansible playbooks for infrastructure automation.

## Structure

- **playbooks/**: Ready-to-run playbooks
  - `system/`: Base OS configuration
  - `services/`: Service deployment
  - `monitoring/`: Monitoring stack setup
- **roles/**: Reusable Ansible roles
- **inventory/**: Host and group definitions
- **tests/**: Syntax and validation tests

## Usage

ansible-playbook -i inventory/hosts playbooks/system/base-setup.yml

## Requirements

- Ansible 2.9+
- SSH access to target hosts

## License

MIT
# Test sync

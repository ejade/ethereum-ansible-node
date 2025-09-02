# Ethereum Node Setup with Ansible

This Ansible playbook sets up an Ethereum node with support for both execution and consensus clients. It includes OS hardening, security configurations, and proper service management.

## Features

- Support for multiple execution clients:
  - Geth
  - Reth
- Support for consensus client:
  - Lighthouse
- OS hardening:
  - UFW firewall configuration
  - Fail2ban setup
  - Chrony NTP configuration
  - Unattended upgrades
- Security features:
  - Dedicated system users for execution and consensus clients
  - JWT authentication between clients
  - Proper file permissions
  - Systemd service management

## Prerequisites

- Ansible 2.9 or later
- Debian-based system (tested on Debian stable)
- Root or sudo access to target machines

## Installation

1. Clone this repository:
```bash
git clone <repository-url>
cd <repository-directory>
```

2. Create an inventory file (`inventory.ini`):
```ini
[ethereum_nodes]
your-node-ip ansible_ssh_user=your-user
```

## Usage

### Basic Setup

To set up a node with default configuration (Geth + Lighthouse on Holesky testnet):
```bash
ansible-playbook -i inventory.ini playbooks/ethereum.yml
```

### Custom Configuration

You can customize the setup using extra variables:

```bash
# Run with Reth instead of Geth
ansible-playbook -i inventory.ini playbooks/ethereum.yml --extra-vars "eth_client=reth"

# Run only execution client (no consensus)
ansible-playbook -i inventory.ini playbooks/ethereum.yml --extra-vars "consensus_client="

# Run only consensus client (no execution)
ansible-playbook -i inventory.ini playbooks/ethereum.yml --extra-vars "eth_client="

# Run on mainnet
ansible-playbook -i inventory.ini playbooks/ethereum.yml --extra-vars "eth_network=mainnet"

# Custom users
ansible-playbook -i inventory.ini playbooks/ethereum.yml \
  --extra-vars "execution_user=myexecution consensus_user=myconsensus"
```

### Available Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `eth_network` | `hoodi` | Network to connect to (mainnet/hoodi) |
| `eth_client` | `geth` | Execution client to use (geth/reth) |
| `consensus_client` | `lighthouse` | Consensus client to use (lighthouse) |
| `execution_user` | `execution` | System user for execution client |
| `execution_group` | `execution` | System group for execution client |
| `consensus_user` | `consensus` | System user for consensus client |
| `consensus_group` | `consensus` | System group for consensus client |

## Service Management

After installation, you can manage the services using systemd:

```bash
# Check status
systemctl status execution
systemctl status consensus

# View logs
journalctl -u execution -f
journalctl -u consensus -f

# Restart services
systemctl restart execution
systemctl restart consensus
```

## Maintenance

The playbook includes maintenance tasks to help manage your Ethereum node:

### Quick Version Check

To quickly check the versions of the installed clients:

```bash
ansible-playbook -i inventory.ini playbooks/version.yml
```

This will display the currently installed versions of both the execution and consensus clients.

### Basic Node Health Check

```bash
ansible-playbook -i inventory.ini ethereum.yml -t status_check
```

This will:
- Display current client versions
- Check service status
- Report disk space usage
- Check sync status of the execution client
- Verify JWT secret file

### Check for Updates

```bash
ansible-playbook -i inventory.ini ethereum.yml -t version_check
```

# Update clients
```bash
ansible-playbook -i inventory.ini ethereum.yml # bear in mind this will upgade if upgrades are available.
```

### Common Maintenance Tasks

```bash
# Restart services
ansible-playbook -i inventory.ini playbooks/ethereum.yml --tags restart <- Not yet implemented

```

## Security Notes

- The playbook creates system users without shell access
- JWT secret is generated securely using OpenSSL and gets proper file permissions
- Firewall rules are configured to allow only necessary ports
- Services run under dedicated users with least privilige principles
- Automatic security updates are enabled (unattened upgrades)

## Directory Structure

```
/secrets/                          # JWT secret storage
/var/lib/execution                 # Execution client data
/var/lib/consensus                 # Consensus client data
```

## Troubleshooting

1. Check service status:
```bash
systemctl status execution
systemctl status consensus
```

2. View logs:
```bash
journalctl -u execution -f
journalctl -u consensus -f
```

3. Check firewall rules:
```bash
ufw status
```

4. Verify JWT secret:
```bash
ls -l /secrets/jwtsecret
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details. 
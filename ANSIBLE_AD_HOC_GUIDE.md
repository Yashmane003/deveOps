# Ansible Ad-Hoc Commands for System Administration

A comprehensive guide on using Ansible ad-hoc commands to perform basic system administration tasks on remote hosts.

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Basic Syntax](#basic-syntax)
4. [Common System Administration Tasks](#common-system-administration-tasks)
5. [Tips and Best Practices](#tips-and-best-practices)

---

## Introduction

Ansible ad-hoc commands are one-liners that allow you to perform quick administrative tasks on remote hosts without creating a playbook. They're perfect for:
- Quick system checks
- One-time configuration changes
- Troubleshooting
- Rapid deployments
- Testing before writing playbooks

Ad-hoc commands use the `ansible` command (not `ansible-playbook`), while playbooks use the `ansible-playbook` command.

---

## Prerequisites

### Requirements
- **Ansible installed** on your control node (the machine from which you run commands)
- **SSH access** to remote hosts (or other connection methods)
- **Inventory file** defining your remote hosts
- **SSH keys** configured for passwordless authentication (recommended)
- **Python** installed on remote hosts

### Installation (Ubuntu/Debian)
```bash
sudo apt-get update
sudo apt-get install ansible
```

### Installation (CentOS/RHEL)
```bash
sudo yum install ansible
```

### Verify Installation
```bash
ansible --version
```

### Basic Inventory Setup

Create `/etc/ansible/hosts` or `~/inventory.ini`:

```ini
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com
db2.example.com

[all_servers:children]
webservers
dbservers
```

---

## Basic Syntax

### General Structure

```bash
ansible <hosts> -m <module> -a '<module_arguments>'
```

### Parameters Explained

| Parameter | Description | Example |
|-----------|-------------|---------|
| `<hosts>` | Target hosts or groups from inventory | `all`, `webservers`, `web1.example.com` |
| `-m, --module-name` | Ansible module to use | `command`, `shell`, `copy`, `apt` |
| `-a, --args` | Arguments passed to the module | `'name=nginx state=present'` |
| `-i, --inventory` | Inventory file path | `-i hosts.ini` |
| `-u, --user` | SSH user | `-u ubuntu` |
| `-b, --become` | Use sudo privilege escalation | `-b` |
| `-k, --ask-pass` | Prompt for SSH password | `-k` |
| `-K, --ask-become-pass` | Prompt for sudo password | `-K` |
| `-C, --check` | Dry-run mode (no changes made) | `-C` |
| `-v, --verbose` | Increase verbosity | `-v` or `-vv` or `-vvv` |
| `--limit` | Limit execution to subset of hosts | `--limit web1.example.com` |
| `-f, --forks` | Number of parallel processes | `-f 5` |

### Basic Examples

```bash
# Ping all hosts in inventory
ansible all -m ping

# Run command on specific host
ansible web1.example.com -m command -a 'whoami'

# Run command on group with sudo
ansible webservers -m command -a 'uptime' -b

# Dry-run to check changes
ansible all -m apt -a 'name=nginx state=present' -C
```

---

## Common System Administration Tasks

### 1. Package Management

#### APT (Debian/Ubuntu)

```bash
# Install a package
ansible all -m apt -a 'name=nginx state=present' -b

# Install multiple packages
ansible all -m apt -a 'name=nginx,curl,vim state=present' -b

# Update package cache
ansible all -m apt -a 'update_cache=yes' -b

# Update all packages (apt upgrade)
ansible all -m apt -a 'name=* state=latest' -b

# Remove a package
ansible all -m apt -a 'name=apache2 state=absent' -b

# Remove package and config files
ansible all -m apt -a 'name=apache2 state=absent purge=yes' -b
```

#### YUM (CentOS/RHEL)

```bash
# Install a package
ansible all -m yum -a 'name=httpd state=present' -b

# Install multiple packages
ansible all -m yum -a 'name=httpd,curl,vim state=present' -b

# Update all packages
ansible all -m yum -a 'name=* state=latest' -b

# Remove a package
ansible all -m yum -a 'name=httpd state=absent' -b
```

---

### 2. System Information Gathering

#### Check System Uptime

```bash
# Simple uptime
ansible all -m command -a 'uptime'

# Using shell for uptime with parsing
ansible all -m shell -a 'uptime | cut -d"," -f1'
```

#### Check Disk Space

```bash
# Disk usage
ansible all -m command -a 'df -h'

# Disk usage for specific mount
ansible all -m command -a 'df -h /'

# Find large files
ansible all -m shell -a 'du -sh /* | sort -rh | head -10'
```

#### Check Memory Usage

```bash
# Memory info
ansible all -m command -a 'free -h'

# Memory and swap usage
ansible all -m shell -a 'free -h && echo "---" && swapon -s'
```

#### Check CPU Information

```bash
# CPU count
ansible all -m command -a 'nproc'

# Detailed CPU info
ansible all -m command -a 'lscpu'

# CPU model
ansible all -m shell -a 'grep "model name" /proc/cpuinfo | head -1'
```

#### Network Information

```bash
# IP addresses
ansible all -m command -a 'ip addr show'

# DNS configuration
ansible all -m command -a 'cat /etc/resolv.conf'

# Network connections
ansible all -m shell -a 'netstat -tuln'
```

#### Gather Facts (Automatic System Information)

```bash
# Gather all facts about hosts
ansible all -m setup

# Gather facts filtered by pattern
ansible all -m setup -a 'filter=ansible_os_family'

# Gather specific fact
ansible all -m setup -a 'filter=ansible_distribution_version'
```

---

### 3. Service Management

#### Start/Stop Services

```bash
# Start a service
ansible webservers -m service -a 'name=nginx state=started' -b

# Stop a service
ansible all -m service -a 'name=nginx state=stopped' -b

# Restart a service
ansible all -m service -a 'name=nginx state=restarted' -b

# Reload service configuration (without restarting)
ansible all -m service -a 'name=nginx state=reloaded' -b

# Enable service to start on boot
ansible all -m service -a 'name=nginx enabled=yes' -b

# Disable service from starting on boot
ansible all -m service -a 'name=nginx enabled=no' -b
```

#### Check Service Status

```bash
# Check if service is running
ansible all -m command -a 'systemctl status nginx' -b

# List all services and their status
ansible all -m command -a 'systemctl list-units --type=service' -b
```

---

### 4. User and Group Management

#### User Management

```bash
# Create a new user
ansible all -m user -a 'name=newuser state=present' -b

# Create user with home directory
ansible all -m user -a 'name=appuser home=/home/appuser createhome=yes' -b

# Create user with specific shell
ansible all -m user -a 'name=webuser shell=/bin/bash' -b

# Create user with sudo privileges
ansible all -m user -a 'name=sudouser groups=sudo append=yes' -b

# Set user password (using hashed password)
ansible all -m user -a 'name=testuser ******' -b

# Delete a user
ansible all -m user -a 'name=olduser state=absent' -b

# Remove user and home directory
ansible all -m user -a 'name=olduser state=absent remove=yes' -b
```

#### Group Management

```bash
# Create a group
ansible all -m group -a 'name=developers state=present' -b

# Add user to group
ansible all -m user -a 'name=john groups=developers,sudo append=yes' -b

# Delete a group
ansible all -m group -a 'name=oldgroup state=absent' -b
```

---

### 5. File Operations

#### Copy Files

```bash
# Copy local file to remote hosts
ansible all -m copy -a 'src=/local/path/file.conf dest=/etc/file.conf' -b

# Copy with owner/permissions
ansible all -m copy -a 'src=/local/file dest=/etc/file owner=root group=root mode=0644' -b

# Copy with backup
ansible all -m copy -a 'src=/local/file dest=/etc/file backup=yes' -b
```

#### Create/Delete Files and Directories

```bash
# Create a file
ansible all -m file -a 'path=/tmp/test.txt state=touch'

# Create a directory
ansible all -m file -a 'path=/var/myapp state=directory mode=0755 owner=appuser' -b

# Delete a file
ansible all -m file -a 'path=/tmp/test.txt state=absent'

# Delete a directory recursively
ansible all -m file -a 'path=/var/myapp state=absent' -b
```

#### Change File Permissions

```bash
# Change permissions
ansible all -m file -a 'path=/etc/myconfig.conf mode=0644' -b

# Change owner and group
ansible all -m file -a 'path=/var/myapp owner=appuser group=appuser' -b

# Change permissions recursively
ansible all -m file -a 'path=/var/myapp state=directory recurse=yes mode=0755' -b
```

#### Create Files with Content

```bash
# Create file with specific content
ansible all -m copy -a 'content="Hello World\n" dest=/tmp/hello.txt'

# Create config file from template/string
ansible all -m shell -a 'echo "server_name example.com;" > /etc/nginx/sites-available/example.com' -b
```

#### File Existence Check

```bash
# Check if file exists
ansible all -m stat -a 'path=/etc/nginx/nginx.conf'

# Check if file exists and get size
ansible all -m command -a 'ls -la /etc/nginx/nginx.conf'
```

---

### 6. Running Arbitrary Commands

#### Command Module (Recommended)

```bash
# Execute a command
ansible all -m command -a 'echo "Hello World"'

# Execute with environment variables
ansible all -m command -a 'env MYVAR=value whoami'

# Change working directory
ansible all -m command -a 'pwd chdir=/tmp'

# Execute as specific user
ansible all -m command -a 'whoami' --become-user=appuser -b
```

#### Shell Module (When Needed)

```bash
# Use shell for pipes and redirects
ansible all -m shell -a 'cat /var/log/syslog | grep nginx'

# Use shell for complex commands
ansible all -m shell -a 'for i in {1..5}; do echo "Line $i"; done'

# Shell with environment variables
ansible all -m shell -a 'echo $HOME'
```

---

### 7. System Reboot and Restart

```bash
# Reboot all hosts
ansible all -m reboot -b

# Reboot with custom timeout (seconds)
ansible all -m reboot -a 'reboot_timeout=600' -b

# Reboot specific group
ansible webservers -m reboot -b

# Schedule reboot after delay
ansible all -m shell -a 'shutdown -r +5' -b
```

---

### 8. Cron Jobs

#### Manage Cron Schedules

```bash
# Add a cron job
ansible all -m cron -a 'name="daily backup" minute="0" hour="2" job="/usr/local/bin/backup.sh"' -b

# Add cron job to run every 30 minutes
ansible all -m cron -a 'name="check app" minute="*/30" job="/usr/local/bin/check.sh"'

# Remove a cron job
ansible all -m cron -a 'name="daily backup" state=absent' -b

# List all cron jobs
ansible all -m shell -a 'crontab -l' -b
```

---

### 9. SSH Key Management

```bash
# Add SSH public key to authorized_keys
ansible all -m authorized_key -a 'user=ubuntu key="{{ lookup(\"file\", \"/home/ubuntu/.ssh/id_rsa.pub\") }}" state=present'

# Remove SSH key from authorized_keys
ansible all -m authorized_key -a 'user=ubuntu key="{{ lookup(\"file\", \"/home/ubuntu/.ssh/id_rsa.pub\") }}" state=absent'
```

---

### 10. Hostname and Network Configuration

#### Change Hostname

```bash
# Set hostname
ansible all -m hostname -a 'name=web01.example.com' -b

# Get current hostname
ansible all -m command -a 'hostname'
```

#### Configure Network Interfaces

```bash
# Bring up network interface
ansible all -m command -a 'ip link set eth0 up' -b

# Bring down network interface
ansible all -m command -a 'ip link set eth0 down' -b

# View network interfaces
ansible all -m command -a 'ip link show'
```

---

### 11. Firewall Management (UFW)

```bash
# Enable firewall
ansible all -m ufw -a 'state=enabled' -b

# Allow port 80
ansible all -m ufw -a 'rule=allow port=80' -b

# Allow specific protocol
ansible all -m ufw -a 'rule=allow proto=tcp port=22' -b

# Deny a port
ansible all -m ufw -a 'rule=deny port=23' -b

# Delete a rule
ansible all -m ufw -a 'rule=allow port=8080 delete=yes' -b

# Check firewall status
ansible all -m command -a 'ufw status' -b
```

---

### 12. SELinux Management (RedHat/CentOS)

```bash
# Check SELinux status
ansible all -m command -a 'getenforce' -b

# Set SELinux to enforcing
ansible all -m selinux -a 'policy=targeted state=enforcing' -b

# Set SELinux to permissive
ansible all -m selinux -a 'policy=targeted state=permissive' -b
```

---

## Tips and Best Practices

### 1. Using Check Mode (Dry-Run)

Test changes before applying them:

```bash
# Preview changes without making them
ansible all -m apt -a 'name=nginx state=present' -C -b

# Verbose check mode to see what changed
ansible all -m apt -a 'name=nginx state=present' -C -b -v
```

### 2. Limit Execution to Specific Hosts

```bash
# Run only on one host
ansible all -m command -a 'uptime' --limit web1.example.com

# Run on hosts excluding certain ones
ansible all -m command -a 'uptime' --limit 'all:!dbservers'

# Run on hosts matching pattern
ansible all -m command -a 'uptime' --limit '*web*'
```

### 3. Run with Increased Verbosity

```bash
# Single verbose (-v)
ansible all -m command -a 'uptime' -v

# Double verbose (-vv) - shows callbacks
ansible all -m command -a 'uptime' -vv

# Triple verbose (-vvv) - shows connections
ansible all -m command -a 'uptime' -vvv

# Very verbose (-vvvv) - everything
ansible all -m command -a 'uptime' -vvvv
```

### 4. Handle Connection Issues

```bash
# Connect with specific SSH user
ansible all -m ping -u ec2-user

# Use specific SSH key
ansible all -m ping --private-key=/path/to/key.pem

# Use password authentication
ansible all -m ping -u ubuntu -k

# Increase connection timeout
ansible all -m ping --ssh-common-args='-o ConnectTimeout=30'
```

### 5. Handle Errors Gracefully

```bash
# Ignore errors and continue
ansible all -m command -a 'false' --ignore-errors

# Get raw output (no JSON parsing)
ansible all -m command -a 'uptime' --raw-output
```

### 6. Parallel Execution

```bash
# Default parallel (5 hosts at a time)
ansible all -m command -a 'uptime'

# Run with 10 parallel processes
ansible all -m command -a 'uptime' -f 10

# Serial execution (one host at a time)
ansible all -m command -a 'uptime' -f 1
```

### 7. Output Formatting

```bash
# Tree format output
ansible all -m command -a 'uptime' -t tree

# JSON output
ansible all -m command -a 'uptime' --raw-output

# Display output for each host
ansible all -m command -a 'uptime' -o
```

### 8. Variable Substitution

```bash
# Pass extra variables
ansible all -m debug -a 'msg="Hello {{ username }}"' -e "username=admin"

# Reference inventory variables
ansible all -m debug -a 'msg="Hostname: {{ inventory_hostname }}"'
```

### 9. Conditional Execution

```bash
# Register output and use conditionally
ansible all -m shell -a 'systemctl is-active nginx' -b

# Check if service is running
ansible all -m command -a 'systemctl status nginx' -b --ignore-errors
```

### 10. Security Best Practices

- ✅ Use SSH keys instead of passwords
- ✅ Use `--ask-pass` or `--ask-become-pass` instead of hardcoding passwords
- ✅ Always use `-b` (become/sudo) for privileged operations
- ✅ Limit execution with `--limit` to specific hosts when testing
- ✅ Use check mode (`-C`) before applying changes
- ✅ Regularly audit your Ansible logs
- ✅ Keep your inventory file secure (restrict permissions)
- ✅ Use Ansible Vault for sensitive data

---

## Real-World Scenarios

### Scenario 1: Deploy and Start a Web Server

```bash
# Update package cache
ansible webservers -m apt -a 'update_cache=yes' -b

# Install Nginx
ansible webservers -m apt -a 'name=nginx state=present' -b

# Start Nginx
ansible webservers -m service -a 'name=nginx state=started enabled=yes' -b

# Verify it's running
ansible webservers -m command -a 'systemctl status nginx' -b
```

### Scenario 2: System Health Check

```bash
# All in one command
ansible all -m shell -a 'echo "=== Uptime ===" && uptime && echo "=== Disk ===" && df -h && echo "=== Memory ===" && free -h'
```

### Scenario 3: Security Update

```bash
# Update all packages
ansible all -m apt -a 'update_cache=yes' -b

# Upgrade packages (with check first)
ansible all -m apt -a 'name=* state=latest' -C -b

# Apply the upgrade
ansible all -m apt -a 'name=* state=latest' -b

# Reboot if needed
ansible all -m reboot -b
```

### Scenario 4: Add New Admin User

```bash
# Create user with sudo access
ansible all -m user -a 'name=adminuser shell=/bin/bash createhome=yes groups=sudo append=yes' -b

# Add SSH key for passwordless login
ansible all -m authorized_key -a 'user=adminuser key="{{ lookup(\"file\", \"/home/current_user/.ssh/id_rsa.pub\") }}" state=present' -b
```

---

## Troubleshooting

### Common Issues

#### SSH Connection Failed
```bash
# Solution 1: Verify host is reachable
ansible all -m ping

# Solution 2: Specify SSH user
ansible all -m ping -u ubuntu

# Solution 3: Use verbose output to diagnose
ansible all -m ping -vvv
```

#### Module Not Found
```bash
# Check available modules
ansible-doc -l

# Get help for specific module
ansible-doc apt
```

#### Permission Denied Errors
```bash
# Use become (sudo)
ansible all -m apt -a 'name=nginx state=present' -b

# Provide sudo password when prompted
ansible all -m apt -a 'name=nginx state=present' -b -K
```

#### Task Timeouts
```bash
# Increase timeout for long-running commands
ansible all -m command -a 'sleep 60' --timeout=120
```

---

## Additional Resources

- [Ansible Official Documentation](https://docs.ansible.com/)
- [Ansible Modules Index](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [Module Documentation](https://docs.ansible.com/ansible/latest/module_plugin_guide.html)

---

**Last Updated:** 2026-06-16
**Author:** DevOps Team

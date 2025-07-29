# Ansible Email Infrastructure with Postfix and OpenDKIM

This repository contains Ansible roles for setting up and configuring email infrastructure components, specifically focused on Postfix with OpenDKIM integration for secure email delivery.

## Repository Structure

```
ansible/
├── README.md                    # This file
└── postfix_opendkim/           # Ansible role for Postfix + OpenDKIM setup
    ├── README.md               # Role-specific documentation
    ├── defaults/               # Default variables
    │   └── main.yml
    ├── files/                  # Static files and templates
    │   └── main.cf.j2         # Postfix main configuration template
    ├── handlers/               # Ansible handlers
    │   └── main.yml
    ├── meta/                   # Role metadata
    │   └── main.yml
    ├── tasks/                  # Main role tasks
    │   └── main.yml
    ├── tests/                  # Test playbooks and inventory
    │   ├── inventory
    │   └── test.yml
    └── vars/                   # Role variables
        └── main.yml
```

## Roles

### postfix_opendkim

A comprehensive Ansible role that automates the installation and configuration of:

- **Postfix**: Modern mail transfer agent (MTA) for secure email delivery
- **OpenDKIM**: DomainKeys Identified Mail (DKIM) implementation for email authentication

#### Features

- Automated installation of Postfix and OpenDKIM packages
- User and group configuration for secure operation
- DKIM key generation with 2048-bit encryption
- Postfix integration with OpenDKIM milter
- DNS TXT record generation for DKIM validation
- Comprehensive logging and debugging support

#### Key Tasks

1. **Package Installation**: Installs `postfix`, `opendkim`, and `opendkim-tools`
2. **User Management**: Adds postfix user to opendkim group for proper permissions
3. **DKIM Configuration**: 
   - Generates DKIM keys with specified domain and selector
   - Configures SigningTable and KeyTable
   - Sets proper file ownership and permissions
4. **DNS Record Generation**: Extracts and displays DKIM DNS TXT record for domain configuration
5. **Postfix Integration**: Configures Postfix to use OpenDKIM milter for message signing

## Prerequisites

- Target systems running RHEL/CentOS (uses `yum` package manager)
- Ansible 2.1 or higher
- Root or sudo access on target hosts
- Valid domain name for DKIM configuration

## Usage

### Basic Playbook Example

```yaml
---
- hosts: mail_servers
  become: yes
  vars:
    domain: "example.com"
    selector: "default"
  roles:
    - postfix_opendkim
```

### Required Variables

- `domain`: Your email domain (e.g., "example.com")
- `selector`: DKIM selector identifier (e.g., "default", "mail")

### Running the Playbook

```bash
# Install the role dependencies
ansible-galaxy install -r requirements.yml

# Run the playbook
ansible-playbook -i inventory site.yml

# Run with specific variables
ansible-playbook -i inventory site.yml -e "domain=yourdomain.com selector=mail"
```

### Testing

```bash
# Test the role locally
cd postfix_opendkim/
ansible-playbook -i tests/inventory tests/test.yml --connection=local
```

## Post-Installation Steps

After running the playbook:

1. **DNS Configuration**: Add the generated DKIM TXT record to your domain's DNS
2. **Firewall**: Ensure port 25 (SMTP) is open if needed
3. **Testing**: Use tools like `opendkim-testkey` to verify DKIM setup
4. **Monitoring**: Check logs in `/var/log/maillog` for proper operation

## Configuration Files

### Postfix Configuration

The role uses a Jinja2 template (`files/main.cf.j2`) based on the standard Postfix configuration with:

- TLS encryption support
- OpenDKIM milter integration
- Security hardening settings
- IPv4/IPv6 dual-stack support

### OpenDKIM Integration

Key configuration parameters added to Postfix:

```
milter_protocol = 2
milter_default_action = accept
smtpd_milters = local:/run/opendkim/opendkim.sock
non_smtpd_milters = local:/run/opendkim/opendkim.sock
```

## Security Considerations

- DKIM keys are generated with 2048-bit encryption
- Proper file permissions and ownership are enforced
- Postfix configured with security best practices
- TLS encryption enabled for secure mail transport

## Troubleshooting

### Common Issues

1. **Permission Errors**: Ensure postfix user is in opendkim group
2. **DKIM Validation Failures**: Verify DNS TXT record is properly configured
3. **Service Start Issues**: Check system logs and service status

### Debugging

Enable debug logging by checking:
- `/var/log/maillog` for Postfix logs
- `/var/log/messages` for OpenDKIM logs
- Use `postconf -n` to verify Postfix configuration

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This project is open source. Please check individual role documentation for specific licensing information.

## Support

For issues and questions:
- Check the troubleshooting section above
- Review system logs for error messages
- Ensure all prerequisites are met
- Verify DNS configuration for DKIM records
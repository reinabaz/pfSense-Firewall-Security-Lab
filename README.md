# pfSense Firewall Security Lab

A comprehensive hands-on network security laboratory for learning firewall configuration, network segmentation, and penetration testing using pfSense, Kali Linux, and Ubuntu in a virtualized environment.

## Project Overview

This lab provides a safe, isolated environment to experiment with network security concepts. You'll learn to configure a pfSense firewall, implement network segmentation, create custom firewall rules, and test their effectiveness through controlled attack simulations.

## Lab Architecture

The lab consists of three virtual machines:

- **pfSense Firewall** - Network gateway and security appliance
- **Kali Linux** - Penetration testing platform (attacker machine)
- **Ubuntu** - Target system (victim machine)

## Prerequisites

- Virtualization platform (VirtualBox recommended)
- Minimum 8GB RAM (12GB+ recommended)
- 100GB+ available disk space
- Basic understanding of networking concepts

## Quick Start

1. **Set up pfSense**: Follow the [pfSense Setup Guide](01-pfsense-setup.md)
2. **Configure Kali Linux**: Follow the [Kali Setup Guide](02-kali-setup.md)
3. **Implement Firewall Rules**: Follow the [Firewall Rules Guide](03-firewall-rules.md)
4. **Run Attack Simulations**: Follow the [Attack Simulations Guide](04-attack-simulations.md)

## Documentation Structure

| Document | Description |
|----------|-------------|
| `01-pfsense-setup.md` | Complete pfSense installation and configuration |
| `02-kali-setup.md` | Kali Linux setup and network configuration |
| `03-firewall-rules.md` | Firewall rule creation and network segmentation |
| `04-attack-simulations.md` | Penetration testing scenarios and validation |

## Learning Objectives

By completing this lab, you will:

- Install and configure pfSense firewall
- Understand network segmentation principles
- Create and manage firewall rules
- Configure network interfaces
- Perform penetration testing with Kali Linux
- Validate security configurations through testing

## Skills Developed

- **Network Security**: Firewall configuration, rule management
- **Penetration Testing**: Attack simulation, vulnerability assessment
- **Network Administration**: Network segmentation
- **Virtualization**: Multi-VM lab environment setup

## Technical Stack

- **pfSense Community Edition** - Open-source firewall platform
- **Kali Linux** - Debian-based penetration testing distribution
- **Ubuntu** - Target operating system
- **VirtualBox** - Virtualization platform

## Security Notice

**This lab is designed for educational purposes only**

- Use only in isolated virtual environments
- Do not apply these techniques against systems you don't own
- Always follow responsible disclosure practices
- Respect applicable laws and regulations

## Contributing

Feel free to:
- Report issues or bugs
- Suggest improvements to the lab setup
- Share additional attack scenarios
- Contribute documentation enhancements

## Resources

- [Kali Linux Documentation](https://www.kali.org/docs/)
- [Network Security Best Practices](https://www.nist.gov/cybersecurity)

---

**Happy learning and stay secure!**

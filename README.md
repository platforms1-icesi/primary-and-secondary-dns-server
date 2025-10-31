# DNS Server Provisioning with Vagrant and Ansible

This project automates the setup of a primary and a secondary DNS server using Infrastructure as Code (IaC) tools. Vagrant is used for creating and managing the virtual machines, and Ansible is used for provisioning the DNS server software (BIND9).

## Project Structure

```
.
├── ansible.cfg
├── deploy.yml
├── inventory.ini
├── README.md
├── roles
│   ├── dns-primary
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │       ├── db.plataformas.local.j2
│   │       ├── named.conf.local.j2
│   │       └── named.conf.options.j2
│   └── dns-secondary
│       ├── handlers
│       │   └── main.yml
│       ├── tasks
│       │   └── main.yml
│       └── templates
│           ├── named.conf.local.j2
│           └── named.conf.options.j2
├── templates
│   └── tsig.key.j2
└── Vagrantfile
```

- **`Vagrantfile`**: Defines the two virtual machines (`dns-primary` and `dns-secondary`) with their respective IP addresses and triggers the Ansible provisioning.
- **`ansible.cfg`**: Ansible configuration file.
- **`inventory.ini`**: Ansible inventory file that defines the hosts to be provisioned.
- **`deploy.yml`**: The main Ansible playbook that maps the roles to the hosts.
- **`roles/`**: This directory contains the Ansible roles for configuring the primary and secondary DNS servers.
  - **`dns-primary/`**: This role configures the primary DNS server. It installs BIND9, sets up the zone files, and configures DNSSEC.
  - **`dns-secondary/`**: This role configures the secondary DNS server. It installs BIND9 and sets it up to replicate the zone from the primary server.
- **`templates/`**: Contains the template for the TSIG key used for secure communication between the primary and secondary servers.

## Infrastructure as Code (IaC)

### Vagrant

Vagrant is used to create a consistent and reproducible development environment. The `Vagrantfile` in this project defines two Ubuntu 22.04 VMs:

- **`dns-primary`**: 192.168.20.10
- **`dns-secondary`**: 192.168.20.11

When you run `vagrant up`, Vagrant automatically creates these VMs and then triggers the Ansible provisioner to configure them.

### Ansible

Ansible is used for configuration management. It automates the process of installing and configuring the DNS servers on the VMs. The configuration is defined in Ansible roles, which are reusable and modular.

## How to Use

1.  **Prerequisites**:
    - [Vagrant](https://www.vagrantup.com/downloads)
    - [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
    - [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

2.  **Clone the repository**:
    ```bash
    git clone https://github.com/platforms1-icesi/primary-and-secondary-dns-server.git
    git clone git@github.com:platforms1-icesi/primary-and-secondary-dns-server.git
    cd primary-and-secondary-dns-server
    ```

3.  **Deploy the environment**:
    ```bash
    vagrant up
    ```

This command will create the VMs and provision them with the DNS server configuration.

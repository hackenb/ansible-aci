# Cisco ACI Automation with Ansible

This repository contains a lab-focused Ansible automation project for configuring a Cisco ACI fabric with a tenant-based application model and external connectivity objects. The playbooks use the Cisco ACI Ansible collection to manage APIC objects such as tenants, VRFs, bridge domains, filters, contracts, application profiles, endpoint groups, and L2Out/L3Out policies.

Note: This repository is just an example how to use a couple of Ansible configurations tested on the Cisco Dcloud Demostration: Get Started with Cisco ACI 6.1. After starting the lab in Dcloud you have to install ansible in WSL first. This is NOT a fully developed Lab - I am working on it that the whole thing makes some sense. Regards h.

## Overview

The project models a sample environment around a tenant named `hacki-tn-a` and provisions a basic multi-tier application structure:

- Network segmentation through VRFs and bridge domains
- Tenant contracts and filters for service communication
- Application profiles and EPGs for web, backend, and database tiers
- VMM domain association for virtual workloads
- L2Out and L3Out policy objects for external connectivity
- OSPF-based routing setup for a small routed lab topology

This is intended primarily for learning, validation, and lab automation rather than production use.

## Topology and configuration model

The project includes several related building blocks:

- Tenant and application data: [conf_ap.yml](conf_ap.yml), [conf_net.yml](conf_net.yml), [conf_contract.yml](conf_contract.yml)
- External VLAN definitions: [vlans.yml](vlans.yml)
- OSPF routing data: [routing.yaml](routing.yaml)
- APIC inventory: [inv.yml](inv.yml)
- Shared APIC connection variables: [group_vars/all.yml](group_vars/all.yml)

Example tenant names and objects used throughout the automation:

- Tenant: `hacki-tn-a`
- VRFs: `hacki-vrf`, `hacki-vrf2`, `hacki-vrf3`
- Bridge domains: `hacki-bd`, `hacki-bd2`
- App profile: `hacki-ap1`
- EPGs: `EPG-web`, `EPG-back`, `EPG-DB`
- L2Out VLAN domains: `L2Out-VLAN-500`, `L2Out-VLAN-505`, `L2Out-VLAN-509`
- L3Out objects: `L3-OSPF-500`, `L3-OSPF-505`, `L3-OSPF-509`

## Repository structure

- [00_config_tenant.yml](00_config_tenant.yml) – Creates the tenant, VRFs, bridge domains, filters, contracts, EPGs, and application policy objects.
- [01_configure_l3_access_policies.yml](01_configure_l3_access_policies.yml) – Creates the L3Out access policy foundation including VLAN pool, L3 domain, AAEP, and domain attachments.
- [02_configure_l2out_vlan_domains.yml](02_configure_l2out_vlan_domains.yml) – Creates L2Out VLAN domains and binds them to the L2Out AAEP and VLAN pool.
- [03_configure_l2out_interface_profile.yml](03_configure_l2out_interface_profile.yml) – Builds the physical L2Out leaf/interface policy structure.
- [04_configure_hacki_tenant_l2out_vlans.yml](04_configure_hacki_tenant_l2out_vlans.yml) – Creates L2Out tenant objects for each VLAN and attaches them to the physical interface.
- [05_configure_hacki_tenant_l3out_routes.yml](05_configure_hacki_tenant_l3out_routes.yml) – Configures OSPF L3Out objects and external interfaces from the routing file.
- [97_delete_tenant_hacki.yml](97_delete_tenant_hacki.yml) – Removes the demo tenant and VMM domain.
- [98_query.yml](98_query.yml) – Queries APIC for tenant inventory.
- [99_create_hacki_admin_user.yml](99_create_hacki_admin_user.yml) – Creates a local lab user named `hacki` with admin privileges.
- [ansible.cfg](ansible.cfg) – Minimal Ansible settings, including the inventory file.
- [requirements.txt](requirements.txt) – Python dependencies used in the project.
- [inv.yml](inv.yml) – APIC host inventory.
- [group_vars/all.yml](group_vars/all.yml) – Shared APIC variables.

## Prerequisites

Before running the playbooks, make sure the following are available:

- Ansible installed on the workstation running the playbooks
- Network reachability to the APIC
- The Cisco ACI collection installed in the Ansible environment

Install the required collection:

```bash
ansible-galaxy collection install cisco.aci
```

The project also defines the APIC connection and TLS behavior in the playbooks:

```yaml
use_ssl: true
validate_certs: false
```

This is convenient for lab use, but it should not be used as-is in a production environment.

## Inventory and credentials

The APIC inventory is defined in [inv.yml](inv.yml):

```yaml
apic:
  hosts:
    apic1.corp.pseudoco.com:
```

The shared variables in [group_vars/all.yml](group_vars/all.yml) contain the APIC connection details:

```yaml
apic_ip: apic1.corp.pseudoco.com
user: "admin"
passw: "C1sco12345"
```

> These values are suitable for a lab demonstration but should be replaced with secure secret management in any real deployment.

## Typical deployment flow

Run the playbooks in the following order:

1. Create the tenant and its base application objects

```bash
ansible-playbook 00_config_tenant.yml
```

2. Configure the L3Out access policy model

```bash
ansible-playbook 01_configure_l3_access_policies.yml
```

3. Create the L2Out VLAN domains

```bash
ansible-playbook 02_configure_l2out_vlan_domains.yml
```

4. Build the physical L2Out interface and leaf model

```bash
ansible-playbook 03_configure_l2out_interface_profile.yml
```

5. Create the tenant-level L2Out objects for each VLAN

```bash
ansible-playbook 04_configure_hacki_tenant_l2out_vlans.yml
```

6. Configure the tenant L3Out OSPF-based external routing objects

```bash
ansible-playbook 05_configure_hacki_tenant_l3out_routes.yml
```

## Useful helper playbooks

- Query APIC tenants

```bash
ansible-playbook 98_query.yml
```

- Remove the demo tenant and VMM domain

```bash
ansible-playbook 97_delete_tenant_hacki.yml
```

- Create the local admin user used for demo access

```bash
ansible-playbook 99_create_hacki_admin_user.yml
```

## Example workflow notes

The project is organized around a simple application contract model:

- `web` EPG consumes or provides key web traffic
- `back` EPG serves internal application traffic
- `db` EPG represents database resources
- Filters and contracts define the allowed TCP ports and service relationships

The YAML data files define this behavior in a parameterized way, making it easy to adjust for a different tenant design or fabric environment.

## Security and operational guidance

This repository stores credentials directly in YAML for simplicity and lab usability. In a production or shared environment, use one of the following instead:

- Ansible Vault
- Environment variables
- A secure secret-management tool
- External credential stores tied into CI/CD pipelines

The playbooks are designed for learning and demonstration and make heavy use of explicit `state: present` and `state: absent` operations to keep the ACI configuration declarative and repeatable.

## Summary

This project demonstrates how Ansible can model and automate Cisco ACI fabric configuration for a realistic tenant scenario. It combines tenant lifecycle operations, Layer 2/Layer 3 networking, filters and contracts, and external routed/bridged connectivity into a compact lab-ready automation set.

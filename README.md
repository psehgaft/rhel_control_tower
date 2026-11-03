# RHEL Control Tower

Ansible playbooks and roles for preparing RHEL 7, 8, 9, and 10 servers. Each role is enabled by a boolean section flag in the inventory.

The starter project includes RHSM registration, approved public repositories, an Identity Management (IdM/FreeIPA) server, IdM users and groups, DNS zones and A records, the IBM AIX Ansible collection, NFS, Podman, OCI Registry v2, HAProxy, Keepalived, administration tools, and OpenSCAP.

> **Compatibility:** This project provides a common automation structure for RHEL 7–10. Package availability and product support depend on the RHEL minor release, architecture, subscription, and configured repositories. RHEL 7 is outside general maintenance; verify current support before production use. Install an IdM server on a clean, dedicated host.

## Requirements and quick start

- Ansible on the control node; SSH, Python, and privilege escalation on managed hosts.
- An active RHSM subscription when using Red Hat content.
- Install the required collections: `ansible-galaxy collection install -r requirements.yml`.
- Edit `inventory/hosts.yml` and `inventory/group_vars/all.yml`. Replace every CHANGEME value and enable only the required section flags.
- Store passwords in an Ansible Vault encrypted file. Do not commit plaintext secrets.
- Review the configuration, then run: `ansible-playbook -i inventory/hosts.yml site.yml --syntax-check`.
- Start with a lab host: `ansible-playbook -i inventory/hosts.yml site.yml --limit rhel_servers`.

## Repository configuration

The `open_repositories` role configures only entries explicitly listed in `open_repositories`. Add approved public repositories with version-appropriate URLs and GPG keys. Red Hat repositories are enabled separately through RHSM using `rhsm_repositories`; the playbook does not enable every available repository automatically.

## Identity Management

The IdM server requires a static FQDN, correct DNS and time synchronization, and prepared firewall/network access. The server provides LDAP and Kerberos identity services. Set `idm_ldap_uri`, `idm_ldap_base_dn`, and `idm_kerberos_realm` for downstream integrations. DNS zones are listed in `idm_dns_zones`; A records use `name`, `zone`, and `ip` in `idm_dns_records`.

The built-in `admin` principal is configured with the IdM admin secret. The local Linux `root` account is intentionally not created or changed by the IdM user role. `sysadmin` and `lector` are domain identities assigned to the configured groups. Set user passwords through an encrypted Vault variable if password management is enabled.

## Security and adding roles

CHANGEME values are placeholders, not usable credentials. Encrypt RHSM, IdM, and Keepalived secrets with Ansible Vault. The registry uses TLS by default; provide valid certificates and persistent storage. OpenSCAP evaluates a selected profile without automatically remediating the host. External CLI binaries require a URL and verified SHA-256 checksum.

To add a role, create `roles/<role_name>/tasks/main.yml`, add a boolean `section_<role_name>` flag to the inventory, and add a conditional `import_role` task to `site.yml`. Document role variables and supported RHEL versions.

## Repository layout

```text
inventory/
  hosts.yml
  group_vars/all.yml
roles/
  <role_name>/tasks/main.yml
site.yml
requirements.yml
ansible.cfg
```

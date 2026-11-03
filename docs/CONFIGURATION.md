# Configuration Guide

Review every variable and test selected roles on disposable systems before production use.

## Section flags

Set a flag to true in inventory/group_vars/all.yml to include its role. All flags default to false.

| Flag | Role | Purpose |
| --- | --- | --- |
| section_subscription | subscription | Register with RHSM and enable explicitly named Red Hat repositories. |
| section_open_repositories | open_repositories | Configure only the public repositories in the allowlist. |
| section_idm_server | idm_server | Install a dedicated IdM server with integrated DNS. |
| section_idm_identity_provider | idm_users_groups | Create IdM groups and users, then assign users to groups. |
| section_idm_dns | idm_dns | Create DNS zones and A records in IdM. |
| section_aix_automation | aix_automation | Install the IBM Power AIX collection on the Ansible control node. |
| section_nfs | nfs | Install NFS utilities, export directories, optionally open firewall services. |
| section_podman | podman | Install Podman and optionally configure registry search settings. |
| section_registry | registry | Run OCI Registry v2 in a Podman container managed by systemd. |
| section_haproxy | haproxy | Render HAProxy frontend and backend definitions. |
| section_keepalived | keepalived | Configure a VRRP virtual IP on the selected interface. |
| section_tools | tools | Install Git, Ansible, Linux Audit Framework, and optional verified binaries. |
| section_openscap | openscap | Install OpenSCAP content and optionally evaluate a selected profile. |

## RHSM and public repositories

Use either RHSM username/password or an organization ID and activation key. Define only repository IDs required by the host in rhsm_repositories.

The open_repositories variable is an allowlist. Each entry requires name, baseurl, and gpgkey. Optional fields include description, gpgcheck, and enabled. Add approved, version-appropriate public repository definitions after checking current upstream documentation. Do not disable GPG checks in production. Official Red Hat repositories are managed separately through RHSM.

## Identity Management server and identity provider

Run the IdM server role on a clean, dedicated host whose hostname matches idm_server_hostname. Set idm_domain, idm_realm, idm_dns_forwarders, and the admin and Directory Manager secrets. The installer also requires correct time synchronization, network connectivity, and DNS planning.

IdM provides LDAP and Kerberos identity services. Set these integration values for applications or clients that consume the directory:

- idm_ldap_uri: LDAPS URI, for example ldaps://idm01.example.com.
- idm_ldap_base_dn: LDAP base distinguished name, for example dc=example,dc=com.
- idm_kerberos_realm: Kerberos realm, usually the uppercase IdM DNS domain.

The built-in admin principal is used for IdM administration. The local Linux root account is outside the IdM directory and is not created or modified by this project. Configure idm_groups and idm_users; each user has name, first, last, and a list of group names. Set idm_user_manage_passwords to true only when user passwords are supplied in an encrypted Vault file.

For DNS, provide zone names in idm_dns_zones. Each idm_dns_records item defines name, zone, and an IPv4 ip. The record name is relative to the zone. Ensure records do not conflict with existing authoritative DNS data.

## Services and security

- NFS: nfs_exports contains the export path, client network, and options. Review client ranges and avoid no_root_squash unless specifically required.
- Podman: podman_registries_conf supports additional registry locations. Avoid insecure registries except in isolated test environments.
- Registry v2: TLS is enabled by default. Install a trusted certificate and key at the configured paths before starting the container. Persist the data directory and plan backups.
- HAProxy: define each frontend with name, bind, and backend. Define each backend with name and a list of servers containing name and address.
- Keepalived: configure interface, VIP, prefix, peer address, VRID, state, and priority for both nodes. Store a strong shared authentication value in Vault.
- Tools: Git, Ansible, and audit packages are selected by RHEL major version. The oc, kubelet, and helm entries are placeholders. Provide compatible download URLs and SHA-256 checksums to install them. A Kubernetes kubelet also requires compatible cluster and runtime configuration; this role does not configure a cluster node.
- OpenSCAP: leave openscap_profile empty to install packages only. Set a profile available in the installed SCAP content to generate a report. This role evaluates compliance and does not apply remediation.

## Ansible Vault workflow

Copy inventory/group_vars/vault.yml.example to inventory/group_vars/vault.yml, replace placeholders, and encrypt it:

    cp inventory/group_vars/vault.yml.example inventory/group_vars/vault.yml
    ansible-vault encrypt inventory/group_vars/vault.yml
    ansible-playbook -i inventory/hosts.yml site.yml --ask-vault-pass

The real vault.yml is ignored by Git. Keep the Vault password outside the repository.

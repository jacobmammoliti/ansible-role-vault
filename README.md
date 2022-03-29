[![Molecule test](https://github.com/jacobmammoliti/ansible-role-vault/actions/workflows/molecule.yml/badge.svg)](https://github.com/jacobmammoliti/ansible-role-vault/actions/workflows/molecule.yml)

# Ansible Role: Vault
A role to deploy a production grade [HashiCorp Vault](https://www.vaultproject.io/).

## Molecule Testing
The following scenarios are currently run with Molecule via GitHub Actions on all pull requests to the `master` branch.

- `default` tests a single Vault install on CentOS 8 with file storage
- `integrated` tests a single Vault install on Ubuntu 20.04 with Integrated storage

## Role Variables
Ansible variables are listed below, along with default values (see `defaults/main.yml`):

### `packer_build`
- If true, Ansible will not copy over certificates, template out configuration files, or start the Vault service. Useful if building a base image with Vault installed
- Default value: `false`

### `use_hashicorp_repository`
- If true, Ansible will install Vault via the official HashiCorp repository. If false, Vault can be installed via remote or local download
- Default value: `false`

### `enterprise`
- If true, Ansible will install the enterprise version of Vault. This is only supported when installing from the HashiCorp repository
- Default value: `false`

### `vault_ansible_group`
- The name of the inventory group
- Default value: `vault`

### `vault_user`
- OS user name
- Default value: `vault`

### `vault_group`
- OS group name
- Default value: `vault`

### `vault_create_account`
- If true, Ansible will create the user and group defined in `vault_user` and `vault_group`
- Default value: `true`

### `vault_home_directory`
- Location of Vault's home directory
- Default value: `/etc/vault.d`

### `vault_data_directory`
- Location of Vault's data directory
- Default value: `/opt/vault`

### `vault_install_directory`
- Location of the Vault binary
- Default value: `/usr/bin`

### `vault_tls_directory`
- Location of the TLS certificates
- Default value: `/opt/vault/tls`

### `vault_license_directory`
- Location of the Vault license
- Default value: `/opt/vault/license`

### `vault_config_file`
- Location of the Vault configuration file
- Default value: `/etc/vault.d/vault.hcl`

### `vault_enable_plugins`
- If true, Ansible will create a plugins directory
- Default value: `false`

### `vault_plugin_directory`
- Location of the plugin directory
- Default value: `/etc/vault.d/plugins`

### `vault_version`
- Version of Vault to install
- Default value: `1.9.1`

### `vault_archive`
- Name of the Vault file archive to download
- Default value: `vault_1.9.1_linux_amd64.zip`

### `vault_download`
- Full remote URL location to the vault archive
- Default value: `https://releases.hashicorp.com/vault/1.9.1/vault_1.9.1_linux_amd64.zip`

### `vault_local_binary_location`
- If set, Ansible will locally look for the Vault binary at the specified path
- No default value

### `vault_license_string`
- If set, Ansible will create a license file using this value for the file content
- No default value. A license is **required** for Vault versions >= 1.8.0 when installing enterprise, so **you have to provide** either `vault_license_string` or `vault_license_file`

### `vault_license_file`
- If set, Ansible will locally look for a Vault license at the specified path
- No default value. A license is **required** for Vault versions >= 1.8.0 when installing enterprise, so **you have to provide** either `vault_license_string` or `vault_license_file`

### `vault_client_addr`
- The address to which Vault will bind client interfaces
- Default value: `0.0.0.0`

### `vault_api_port`
- Vault API port
- Default value: `8200`

### `vault_cluster_name`
- The name of the Vault cluster
- Default value: none

### `vault_cluster_port`
- Vault cluster port
- Default value: `8201`

### `vault_ui_enable`
- If true, Ansible will enable the Vault UI
- Default value: `true`

### `cloud`
- The cloud provider Vault will be deployed to. This is used for Raft's auto-join feature. Refer to [this](https://www.consul.io/docs/install/cloud-auto-join#provider-specific-configurations) document to determine what keys are needed for each cloud environment. On-prem/VMWare installs would set this to `none`
- Type: `dictionary`
- Default value: `provider: 'none'`

### `vault_storage_backend`
- Storage backend to use (supports file, integrated, or Consul)
- Default value: `integrated`

### `vault_telemetry`
- Specifies telemetry configuration to Vault. Refer to [this](https://www.vaultproject.io/docs/configuration/telemetry#telemetry-parameters) document for available parameters. Each parameter can be passed as a key to this dictionary
- No default value

### `vault_seal`
- A dictionary defining the seal type to use for Vault. Refer to [this](https://www.vaultproject.io/docs/configuration/seal) document for the required parameters to pass as keys if using a cloud KMS solution.
- Type: `dictionary`
- Default value: `type: 'shamir'`

### `vault_tls_disable_client_certs`
- If true, Vault will not request for a client certificate when accessed
- Default value: true

### `vault_tls_ca_cert_file`
- If set, Ansible will locally look for a TLS CA certificate at the specified path to copy to each Vault server. Mutually exclusive with `vault_tls_ca_cert_string`
- No default value

### `vault_tls_ca_cert_string`
- If set, Ansible will Ansible will create the TLS CA file using this value for the file content. Mutually exclusive with `vault_tls_ca_cert_file`
- No default value

### `vault_tls_cert_file`
- If set, Ansible will locally look for a TLS certificate at the specified path to copy to each Vault server. Mutually exclusive with `vault_tls_cert_string`
- No default value

### `vault_tls_cert_string`
- If set, Ansible will Ansible will create the TLS certificate using this value for the file content. Mutually exclusive with `vault_tls_cert_file`
- No default value

### `vault_tls_key_file`
- If set, Ansible will locally look for a TLS key at the specified path to copy to each Vault server. Mutually exclusive with `vault_tls_key_string`
- No default value

### `vault_tls_key_string`
- If set, Ansible will Ansible will create the TLS key using this value for the file content. Mutually exclusive with `vault_tls_cert_file`
- No default value

### `vault_leader_tls_servername`
- If Set, Vault will use this TLS server name when connecting with HTTPS. Uses the server's hostname if not set.
- No default value

### `consul_http_port`
- Port to use to connect to the local Consul agent
- Default value: `8500`

### `consul_scheme`
- Scheme to use to connect to the local Consul agent
- Default value: `http`

### `consul_vault_acl_token`
- Consul ACL token that Vault should use
- No default value

### `consul_tls_directory`
- Location of the TLS certificates for Consul
- Default value: `/opt/vault/consul`

### `consul_tls_ca_file`
- If set, Ansible will locally look for a TLS CA certificate at the specified path to copy to each Vault server for connecting to Consul
- No default value

### `consul_tls_cert_file`
- If set, Ansible will locally look for a signed TLS certificate at the specified path to copy to each Vault server for connecting to Consul
- No default value

### `consul_tls_key_file`
- If set, Ansible will locally look for a TLS key at the specified path to copy to each Vault server for connecting to Consul
- No default value

### `consul_tls_skip_verify`
- If true, Vault will skip verification of the certificate presented by Consul
- Default value: `false`

### `consul_vault_kv_path`
- Path to Vault's data in Consul's key value storage
- Default value: `vault/`

## Dependencies
- Consul cluster if using Consul as backend storage.

## Example with Integrated Storage
The following example deploys a three node Vault cluster with Integrated storage.

> Using Google Cloud Platform for this example.

```bash
$ export GOOGLE_CLOUD_PROJECT=<your_gcp_project_id>

# creation a bastion node to run playbook from
$ gcloud compute instances create bastion \
  --async \
  --zone us-central1-a \
  --boot-disk-size 50GB \
  --image-family ubuntu-2004-lts \
  --image-project ubuntu-os-cloud \
  --machine-type n1-standard-1 \
  --tags vault

# create three nodes in GCP
$ for i in 1 2 3; do
  gcloud compute instances create vault0${i} \
    --async \
    --zone us-central1-a \
    --boot-disk-size 50GB \
    --image-family ubuntu-2004-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --tags vault
done

# SSH to bastion node
$ gcloud compute ssh bastion --zone us-central1-a

# optional: generate SSL certificates
$ consul tls ca create
==> Saved consul-agent-ca.pem
==> Saved consul-agent-ca-key.pem

$ consul tls cert create -server \
  -additional-dnsname=vault01.c.$GOOGLE_CLOUD_PROJECT.internal \
  -additional-dnsname=vault02.c.$GOOGLE_CLOUD_PROJECT.internal \
  -additional-dnsname=vault03.c.$GOOGLE_CLOUD_PROJECT.internal
...
==> Using consul-agent-ca.pem and consul-agent-ca-key.pem
==> Saved dc1-server-consul-0.pem
==> Saved dc1-server-consul-0-key.pem

# create the inventory file
$ cat <<EOF > inventory
[vault_primary]
vault01.c.$GOOGLE_CLOUD_PROJECT.internal
vault02.c.$GOOGLE_CLOUD_PROJECT.internal
vault03.c.$GOOGLE_CLOUD_PROJECT.internal
EOF

# create the playbook
$ cat <<EOF > site.yml
---
- hosts: vault_primary
  become: yes
  roles:
    - role: ansible-role-vault
EOF

# optional: create group_vars directory and file
$ mkdir group_vars

$ cat <<EOF > vault_primary.yml
---
vault_storage_backend: 'integrated'
vault_version: '1.9.0+ent'

vault_tls_ca_cert_file: 'tls/consul-agent-ca.pem'
vault_tls_cert_file: 'tls/dc1-server-consul-0.pem'
vault_tls_key_file: 'tls/dc1-server-consul-0-key.pem'
vault_tls_disable_client_certs: false

vault_ansible_group: vault_primary

vault_seal:
  type: gcpkms
  project: $GOOGLE_CLOUD_PROJECT
  region: us-central1
  key_ring: vault_key_ring
  crypto_key: vault_crypto_key
EOF

# run ansible playbook
$ ansible-playbook -i inventory site.yaml

# export address for binary
$ export VAULT_ADDR=https://vault01.c.[PROJECT_ID].internal:8200

# optional: disable SSL verification
$ export VAULT_SKIP_VERIFY=true

# initalize the Vault cluster (if using Shamir)
$ vault operator init -key-shares=1 -key-threshold=1
Unseal Key 1: tMoFtiYOuBlf6757jjOl4lCvN1v4NneZhzQqwe3pzxA=

Initial Root Token: s.8OZR9fj3g3mJoxDKlUaE48Yx

Success! Vault is initialized
...

# initalize the Vault cluster (if using Autounseal)
$ vault operator init -recovery-shares=1 -recovery-threshold=1
Recovery Key 1: tMoFtiYOuBlf6757jjOl4lCvN1v4NneZhzQqwe3pzxA=

Initial Root Token: s.8OZR9fj3g3mJoxDKlUaE48Yx

Success! Vault is initialized
...

# unseal each Vault node if using Shamir
$ vault operator unseal tMoFtiYOuBlf6757jjOl4lCvN1v4NneZhzQqwe3pzxA

# authenticate to Vault with the root token
$ vault login s.8OZR9fj3g3mJoxDKlUaE48Yx
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.
...

# verify raft peer list
$ vault operator raft list-peers
Node                               Address                                 State       Voter
----                               -------                                 -----       -----
vault01.c.[PROJECT_ID].internal    vault01.c.[PROJECT_ID].internal:8201    leader      true
vault02.c.[PROJECT_ID].internal    vault02.c.[PROJECT_ID].internal:8201    follower    true
vault03.c.[PROJECT_ID].internal    vault03.c.[PROJECT_ID].internal:8201    follower    true
```

Author Information
------------------
Jacob Mammoliti

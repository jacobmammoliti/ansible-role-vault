[![Molecule test](https://github.com/jacobmammoliti/ansible-role-vault/actions/workflows/molecule.yml/badge.svg)](https://github.com/jacobmammoliti/ansible-role-vault/actions/workflows/molecule.yml)

# Ansible Role: Vault
A role to deploy a production grade [HashiCorp Vault](https://www.vaultproject.io/).

## Molecule Testing
The following scenarios are currently run with Molecule via GitHub Actions on all pull requests to the `master` branch.

- `default` tests a single Vault install on CentOS 8 with file storage
- `integrated` tests a single Vault install on Ubuntu 20.04 with Integrated storage

## Role Variables
Ansible variables are listed below, along with default values (see `defaults/main.yml`):

### `vault_user`

- OS user
- Default value: `vault`

### `vault_group`

- OS group
- Default value: `vault`

### `vault_create_account`

- Whether to create the user and group defined by `vault_user` and `vault_group` or not
- Default value: `true`

### `vault_home_directory`

- Location of Vault's home directory
- Default value: `/etc/vault.d`

### `vault_data_directory`

- Location of Vault's data directory
- Default value: `/opt/vault`

### `vault_install_directory`

- Location of the Vault binary
- Default value: `/opt/vault/bin`

### `vault_config_file`

- Location of the Vault configuration file
- Default value: `/etc/vault.d/vault.hcl`

### `vault_plugin_directory`

- Location of Vault's plugin directory
- Default value: `/etc/vault.d/plugins`

### `vault_enable_plugins`

- Whether to create the plugins directory or not
- Default value: `true`

### `vault_version`

- Version of Vault to download and install
- Default value: `1.9.0`

### `vault_archive`

- Name of the Vault file archive to download
- Default value: `vault_1.9.0_linux_amd64.zip`

### `vault_download`

- Full URL location to download vault
- Default value: `https://releases.hashicorp.com/vault/1.9.0/vault_1.9.0_linux_amd64.zip`

### `vault_local_binary_location`

- Location of the local binary
- Default value: `None`

### `vault_vault_license_path`

- Location of the Vault license file
- Default value: `/opt/vault/license/vault.hclic`

### `vault_client_addr`

- The address to which Vault will bind client interfaces
- Default value: `0.0.0.0`

### `vault_http_port`

- HTTP port for Vault
- Default value: `8200`

### `vault_api_port`

- HTTP API port for Vault
- Default value: `8201`

### `vault_ui_enable`

- Whether the UI is enabled or not
- Default value: `true`

### `vault_storage_backend`

- Storage backend to use
- Default value: `integrated`

### `vault_telemetry`

- Dictionary containing telemetry key-value data
- Default value: `None`

### `vault_seal`

- Seal type to use
- Type: `dictionary`
- Default value: `type: 'shamir'`

### `vault_tls_disable`

- Whether to disable TLS or not
- Default value: `true`

### `vault_tls_directory`

- Directory that TLS certificates live in
- Default value: `/opt/vault/tls`

### `vault_tls_ca_cert_file`

- Local path to the TLS CA certificate to copy over
- Default value: `None`

### `vault_tls_cert_file`

- Local path to the TLS signed certificate to copy over
- Default value: `None`

### `vault_tls_key_file`

- Local path to the TLS key to copy over
- Default value: `None`

### `vault_tls_disable_client_certs`

- Whether to disable client certificates or not
- Default value: `true`

### `cloud`

- The cloud provider Vault will be deployed to
- Type: `dictionary`
- Default value: `provider: 'none'`

### `enterprise`
- Whether to install enterprise Vault or not (only supported when using the repository method)
- Default value: `false`

### `consul_http_port`

- Port to use to connect to local Consul agent
- Default value: `8500`

### `consul_scheme`

- Scheme to use to connect to Consul
- Default value: `http`

### `consul_vault_kv_path`

- Path to Vault's data in Consul KV
- Default value: `vault/`

### `consul_acl_enabled`

- Whether Consul ACLs are enabled or not
- Default value: `false`

### `consul_tls_directory`

- Directory that Consul TLS certificates live in
- Default value: `None`

### `consul_tls_ca_file`

- Local path to the TLS CA certificate to copy over
- Default value: `None`

### `consul_tls_cert_file`

- Local path to the TLS signed certificate to copy over
- Default value: `None`

### `consul_tls_key_file`

- Local path to the TLS key to copy over
- Default value: `None`

### `consul_tls_skip_verify`

- Whether or not to skip verification of Consul TLS certificates
- Default value: `false`

### `consul_vault_acl_token`

- Vault's ACL token in Consul (requred if ACLS are enabled in Consul cluster)
- Default value: `None`

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
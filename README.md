[![Molecule test](https://github.com/jacobmammoliti/ansible-role-vault/actions/workflows/molecule.yml/badge.svg)](https://github.com/jacobmammoliti/ansible-role-vault/actions/workflows/molecule.yml)

# Ansible Role: Vault
A role to deploy a production grade [HashiCorp Vault](https://www.vaultproject.io/).

## Role Variables
Ansible variables are listed below, along with default values (see `defaults/main.yml`):

### `vault_user`

- OS user
- Default value: vault

### `vault_group`

- OS group
- Default value: vault

### `vault_create_account`

- Whether to create the user and group defined by `vault_user` and `vault_group` or not
- Default value: true

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

### `vault_version`

- Version of Vault to download and install
- Default value: `1.6.3`

### `vault_archive`

- Name of the Vault file archive to download
- Default value: `vault_1.6.3_linux_amd64.zip`

### `vault_download`

- Full URL location to download vault
- Default value: `https://releases.hashicorp.com/vault/1.6.3/vault_1.6.3_linux_amd64.zip`

### `vault_local_binary`

- Whether to use a binary stored locally (this is mutually exclusive with the three above variables)
- Default value: false

### `vault_local_binary_location`

- Location of the local binary
- Default value: `binary/vault`

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
- Default value: true

### `vault_storage_backend`

- Storage backend to use
- Default value: file

### `vault_telemetry`

- Dictionary containing telemetry key-value data
- Default value: None

### `vault_disable_mlock`

- Whether to disable memory lock or not
- Default value: false

### `vault_seal_type`

- Seal type to use
- Default value: shamir

### `vault_seal_gcp_project`

- GCP project the KMS lives in
- Default value: None

### `vault_seal_gcp_region`

- GCP region the KMS lives in
- Default value: None

### `vault_seal_gcp_key_ring`

- Name of the KMS Keyring in GCP
- Default value: None

### `vault_seal_gcp_crypto_key`

- Name of the KMS Key in GCP
- Default value: None

### `vault_seal_aws_region`

- AWS region the KMS lives in
- Default value: None

### `vault_seal_aws_access_key`

- Name of the AWS access key
- Default value: None

### `vault_seal_aws_secret_key`

- Name of the AWS secret key
- Default value: None

### `vault_seal_aws_kms_key_id`

- AWS KMS key ID
- Default value: None

### `vault_seal_aws_endpoint`

- (optional) AWS KMS API endpoint. If not set, default API endpoint for region will be used.
- Default value: None
- [Vault API ref](https://www.vaultproject.io/docs/configuration/seal/awskms#endpoint)

### `vault_hsm_crypto_library`

- Path to the HSM's local crypto library
- Default value: None

### `vault_hsm_slot`

- The HSM slot number
- Default value: None

### `vault_hsm_pin`

- The HSM pin Vault will use to connect
- Default value: None

### `vault_hsm_generate_key`

- Whether or not Vault to generate its own encryption key in the HSM
- Default value: None

### `vault_hsm_key_label`

- The encryption key label for Vault in the HSM
- Default value: None

### `vault_hsm_hmac_key_label`

- The encryption key HMAC label for Vault in the HSM
- Default value: None

### `vault_tls_disable`

- Whether to disable TLS or not
- Default value: true

### `vault_tls_directory`

- Directory that TLS certificates live in
- Default value: `/etc/vault.d/tls`

### `vault_tls_cert_file`

- Local path to the TLS signed certificate to copy over
- Default value: `tls/dc1-server-consul-0.pem`

### `vault_tls_key_file`

- Local path to the TLS key to copy over
- Default value: `tls/dc1-server-consul-0-key.pem`

### `vault_tls_disable_client_certs`

- Whether to disable client certificates or not
- Default value: true

### `consul_http_port`

- Port to use to connect to local Consul agent
- Default value: `8500`

### `consul_scheme`

- Scheme to use to connect to Consul
- Default value: http

### `consul_vault_kv_path`

- Path to Vault's data in Consul KV
- Default value: `vault/`

### `consul_acl_enabled`

- Whether Consul ACLs are enabled or not
- Default value: false

### `consul_tls_directory`

- Directory that Consul TLS certificates live in
- Default value: `/etc/consul.d/tls`

### `consul_tls_ca_file`

- Local path to the TLS CA certificate to copy over
- Default value: `tls/consul-agent-ca.pem`

### `consul_tls_cert_file`

- Local path to the TLS signed certificate to copy over
- Default value: `tls/dc1-server-consul-0.pem`

### `consul_tls_key_file`

- Local path to the TLS key to copy over
- Default value: `tls/dc1-server-consul-0-key.pem`

### `consul_tls_skip_verify`

- Whether or not to skip verification of Consul TLS certificates
- Default value: false

### `consul_vault_acl_token`

- Vault's ACL token in Consul (requred if ACLS are enabled in Consul cluster)
- Default value: None

## Dependencies
- Consul cluster if using Consul as backend storage.

## Example with Integrated Storage
The following example deploys a three node Vault cluster with Integrated storage.

> Using Google Cloud Platform for this example.

```bash
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
$ for i in 0 1 2; do
  gcloud compute instances create vault-primary-${i} \
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

# create the inventory file
$ cat <<EOF > inventory
[vault]
vault-0.c.[PROJECT_ID].internal
vault-1.c.[PROJECT_ID].internal
vault-2.c.[PROJECT_ID].internal
EOF

# verify connectivity to each compute instance
$ ansible -i inventory vault -m ping
vault-primary-0.c.[PROJECT_ID].internal | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
vault-primary-1.c.[PROJECT_ID].internal | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
vault-primary-2.c.[PROJECT_ID].internal | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

# run ansible playbook
$ ansible-playbook -i inventory site.yaml

# export address for binary
$ export VAULT_ADDR=http://vault-primary-0.c.[PROJECT_ID].internal:8200

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

# authenticate to Vault with the root token
$ vault login s.8OZR9fj3g3mJoxDKlUaE48Yx
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.
...

# verify raft peer list
$ vault operator raft list-peers
Node                                       Address                                         State       Voter
----                                       -------                                         -----       -----
vault-primary-0.c.[PROJECT_ID].internal    vault-primary-0.c.[PROJECT_ID].internal:8201    leader      true
vault-primary-1.c.[PROJECT_ID].internal    vault-primary-1.c.[PROJECT_ID].internal:8201    follower    true
vault-primary-2.c.[PROJECT_ID].internal    vault-primary-2.c.[PROJECT_ID].internal:8201    follower    true
```

Author Information
------------------
Jacob Mammoliti
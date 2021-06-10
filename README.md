[![Build Status](https://travis-ci.org/arctiqjacob/ansible-role-vault.svg?branch=master)](https://travis-ci.org/arctiqjacob/ansible-role-vault)

Ansible Role: Vault
===================
A role to deploy a production grade [HashiCorp Vault](https://www.vaultproject.io/).

Role Variables
--------------
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

### `vault_ui`

- Whether the UI is enabled or not
- Default value: true

### `vault_storage_backend`

- Storage backend to use
- Default value: file

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

### `consul_tls_enabled`

- Whether Consul TLS is enabled or not
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

Dependencies
------------
- Consul cluster if using Consul as backend storage.

Example with Integrated Storage on GCP
--------------------------------------
The following example deploys a three node Vault 1.5 cluster with integrated storage in Google Cloud Platform.

Create three compute instances which will host the Vault servers:
```shell
for i in 0 1 2; do
  gcloud compute instances create vault-${i} \
    --async \
    --no-address \
    --boot-disk-size 100GB \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --scopes=https://www.googleapis.com/auth/cloudkms \
    --tags vault
done
```

Create an inventory file:
```shell
$ cat > inventory <<EOF
[vault]
vault-0.c.[PROJECT_ID].internal
vault-1.c.[PROJECT_ID].internal
vault-2.c.[PROJECT_ID].internal
EOF
```

Ensure Python is installed on the servers:
```shell
$ ansible -i inventory vault -m ping
vault-0.c.[PROJECT_ID].internal | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
vault-2.c.[PROJECT_ID].internal | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
vault-1.c.[PROJECT_ID].internal | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

Run the Ansible playbook:
```shell
$ ansible-playbook -i inventory site.yaml
...
PLAY RECAP *********************************************************************
vault-0.c.[PROJECT_ID].internal : ok=11   changed=9    unreachable=0    failed=0   
vault-1.c.[PROJECT_ID].internal : ok=11   changed=9    unreachable=0    failed=0   
vault-2.c.[PROJECT_ID].internal : ok=11   changed=9    unreachable=0    failed=0  
```

Initialize Vault:
```shell
$ export VAULT_ADDR=http://vault-0.c.[PROJECT_ID].internal:8200

$ vault operator init -recovery-shares=1 -recovery-threshold=1
Recovery Key 1: tMoFtiYOuBlf6757jjOl4lCvN1v4NneZhzQqwe3pzxA=

Initial Root Token: s.8OZR9fj3g3mJoxDKlUaE48Yx

Success! Vault is initialized

Recovery key initialized with 1 key shares and a key threshold of 1. Please
securely distribute the key shares printed above.

$ vault login
Token (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.
...

$ vault operator raft list-peers
Node      Address                                    State       Voter
----      -------                                    -----       -----
vault0    vault-0.c.[PROJECT_ID].internal:8201       leader      true
vault1    vault-1.c.[PROJECT_ID].internal:8201       follower    true
vault2    vault-2.c.[PROJECT_ID].internal:8201       follower    true
```

Example with Consul Storage on GCP
----------------------------------
The following example deploys a three node Vault 1.4 cluster with Consul storage in Google Cloud Platform.

Create three compute instances which will host the Consul servers and two compute instances which will host the Vault servers:
```shell
for i in 0 1 2; do
  gcloud compute instances create consul-${i} \
    --async \
    --no-address \
    --boot-disk-size 100GB \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --scopes=compute-ro \
    --tags consul
done

for i in 0 1; do
  gcloud compute instances create vault-${i} \
    --async \
    --no-address \
    --boot-disk-size 100GB \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --scopes=https://www.googleapis.com/auth/cloudkms, compute-ro \
    --tags vault
done
```

Create an inventory file:
```shell
cat > inventory <<EOF
[vault]
[servers]
consul-0.c.[PROJECT_ID].internal
consul-1.c.[PROJECT_ID].internal
consul-2.c.[PROJECT_ID].internal

[clients]
vault-0.c.[PROJECT_ID].internal
vault-1.c.[PROJECT_ID].internal

[vault]
vault-0.c.[PROJECT_ID].internal
vault-1.c.[PROJECT_ID].internal
EOF
```

Create an Ansible playbook, calling both roles:
```shell
cat > main.yaml <<EOF
---
- hosts: servers, clients
  become: yes
  roles:
    - role: ansible-role-consul

- hosts: vault
  become: yes
  roles:
    - role: ansible-role-vault
      vars:
        vault_config:
          client_addr: '0.0.0.0'
          tls_disable: false
          tls_cert_file: 'vault.crt'
          tls_key_file: 'vault.key'
          http_port: '8200'
          api_port: '8201'
          ui: true
          storage: 'consul'
        seal:
          type: 'gcpkms'
          project: '[PROJECT_ID]'
          region: 'global'
          key_ring: '[VAULT_KEY_RING]'
          crypto_key: '[VAULT_KEY]'
EOF
```

Ensure Python is installed on the servers:
```shell
$ ansible -i inventory vault -m ping
vault-1.c.[PROJECT_ID].internal | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
vault-0.c.[PROJECT_ID].internal | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
consul-2.c.[PROJECT_ID].internal | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
consul-1.c.[PROJECT_ID].internal | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
consul-0.c.[PROJECT_ID].internal | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

Run the Ansible playbook:
```shell
$ ansible-playbook -i inventory main.yaml
...
PLAY RECAP *********************************************************************
consul-0.c.blizzard-253119.internal : ok=11   changed=2    unreachable=0    failed=0   
consul-1.c.blizzard-253119.internal : ok=11   changed=2    unreachable=0    failed=0   
consul-2.c.blizzard-253119.internal : ok=11   changed=2    unreachable=0    failed=0   
vault-0.c.blizzard-253119.internal : ok=20   changed=2    unreachable=0    failed=0   
vault-1.c.blizzard-253119.internal : ok=20   changed=2    unreachable=0    failed=0 
```

Initialize Vault:
```shell
$ export VAULT_ADDR=http://vault-0.c.[PROJECT_ID].internal:8200
$ export CONSUL_HTTP_ADDR=http://consul-0.c.[PROJECT_ID].internal:8500

$ vault operator init -recovery-shares=1 -recovery-threshold=1
Recovery Key 1: tMoFtiYOuBlf6757jjOl4lCvN1v4NneZhzQqwe3pzxA=

Initial Root Token: s.8OZR9fj3g3mJoxDKlUaE48Yx

Success! Vault is initialized

Recovery key initialized with 1 key shares and a key threshold of 1. Please
securely distribute the key shares printed above.

$ vault login
Token (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.
...
```

Author Information
------------------
Jacob Mammoliti | jacob.mammoliti@arctiq.ca

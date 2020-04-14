Ansible Role: Vault
===================

A role to deploy a production grade [HashiCorp Vault](https://www.vaultproject.io/).

Requirements
------------

- Compute Admin role in a GCP Project

Role Variables
--------------

Ansible variables are listed below, along with default values (see `defaults/main.yml`):

Controls whether a separate account is created or not and what the user and group should be named.
```yaml
vault_user: 'vault'
vault_group: 'vault'
vault_create_account: true
```

Where to initialize Vault's home, data, and install directory.
```yaml
vault_home_directory: '/etc/vault.d'
vault_data_directory: '/opt/vault'
vault_install_directory: '{{ vault_data_directory }}/bin'
vault_config_file: '{{ vault_home_directory }}/vault.hcl'
```

The version of Vault to install and where it should download its binary from.
```yaml
vault_version: '1.4.0'
vault_archive: 'vault_{{ vault_version }}_linux_amd64.zip'
vault_download: 'https://releases.hashicorp.com/vault/{{ vault_version }}/{{ vault_archive }}'
```

Controls how Vault should be configured. This data is used in the templated out `vault.hcl` file.
```yaml
vault_config:
  client_addr: '0.0.0.0'
  tls_disable: true
  tls_cert_file: ''
  tls_key_file: ''
  http_port: '8200'
  api_port: '8201'
  ui: true
  storage: 'file'
```

Controls how Vault's sealing mechanism. This data is used in the templated out `vault.hcl` file.
```yaml
seal:
  type: ''
  project: ''
  region: ''
  key_ring: ''
  crypto_key: ''
```

Dependencies
------------

None.

Example
-------

The following example deploys a three node Vault 1.4 cluster with integrated storage in Google Cloud Platform.

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
cat > inventory <<EOF
[vault]
vault-0.c.[PROJECT_ID].internal
vault-1.c.[PROJECT_ID].internal
vault-2.c.[PROJECT_ID].internal
EOF
```

Create an Ansible playbook, calling the role:
```shell
cat > main.yaml <<EOF
---
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
          storage: 'raft'
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
ansible -i inventory vault -m ping
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
$ ansible-playbook -i inventory main.yaml
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

Author Information
------------------

Jacob Mammoliti | jacob.mammoliti@arctiq.ca
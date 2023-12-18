hashicorp-vault-password-rotation
=========

This role sets up password rotation for the specified user/s using the script and idea from [painless-password-rotation](https://github.com/scarolan/painless-password-rotation)  
The role will always create a new server token when it is used and therefore it should only be run during onboarding or when changes are required  


It is a good idea to use a token role for the server token creation and to give the ansible host only permissions to use this role as this is a more secure way of handling the token creation  
Here is an example:
```
# Create the token role
vault write auth/token/roles/rotate-linux allowed_policies=rotate-linux-policy name=rotate-linux orphan=true renewable=true period=24h
# Give access to the token role in a policy
# ansible-policy
path "auth/token/create/rotate-linux" {
  capabilities = ["create", "read", "update"]
}
```

Requirements
------------
- hashicorp vault setup and reachable
- vault plugin [vault-secrets-gen](https://github.com/sethvargo/vault-secrets-gen) working
- 2k/v secret backend mounted at systemcreds
- hvac pip plugin installed on the ansible host

Role Variables
--------------
- `vault_pw_rot_token_ttl`: [optional]: Maximum time the server token will live until it will expire 
- `vault_pw_rot_base_path`: [default: `/opt/password-rotation`]: Basepath for the environment file and script file 
- `vault_pw_rot_env_file`: [default: `'{{ vault_pw_rot_base_path }}/rotate_linux_password.env'`]: Path to store the environment file
- `vault_pw_rot_script_file`: [default: `'{{ vault_pw_rot_base_path }}/rotate_password.sh'`]: Path to store the script file
- `vault_pw_rot_script_flags`: [default: `'-t password -l 20 -d 5 -s 0'`]: Default flags for the script, check [painless-password-rotation](https://github.com/scarolan/painless-password-rotation) for details
- `vault_pw_rot_time`: [default: `12h`]: Default time until the systemd trigger runs the password rotation
- `vault_pw_rot_auth_method`: [default: `token`]: Authentification method used to create the server token
- `vault_pw_rot_url`: [required]: URL to the vault server
- `vault_pw_rot_token`: [required]: Vault token to use to create the server token, this can also be set differently, check the ansible-collections-community-hashi-vault-vault-token-create-module for details
- `vault_pw_rot_token_namespace`: [optional]: Vault namespace to create the token for
- `vault_pw_rot_token_policies`: [optional]: Subset of policies for the server token
- `vault_pw_rot_token_role`: [optional]: Token role used to create the server token
- `vault_pw_rot_users`: [required]: List of dictionaries to specify which users should get their passwords rotated as well as how or when.
Example:
```
vault_pw_rot_users:
  - name: root
  - name: admin
    flags: '-t passphrase -w 10'
    time: 5h
```

Dependencies
------------
- [0x0i.systemd](https://galaxy.ansible.com/ui/standalone/roles/0x0I/systemd/)
- [hashi_vault collection](https://galaxy.ansible.com/ui/repo/published/community/hashi_vault)

Example Playbook
----------------
```
    - hosts: servers
      vars:
        vault_pw_rot_url: 'https://vaultserver.domain.com'
        vault_pw_rot_token: 'xxxxxxxxxx-xxxxxx'
        vault_pw_rot_users:
          - name: root
      roles:
        - hashicorp-vault-password-rotation
```
Author Information
------------------

[c-erb](https://github.com/c-erb)  
[PierreGode](https://github.com/PierreGode)

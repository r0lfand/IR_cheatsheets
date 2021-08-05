# Account manipulation: SSH Authorized Keys T1098.004
## Persistence with SSH key playbook
1. If an account has been compromised it has to be locked.
2. Kill user's active sessions.
3. Check if any keys were added. Contact the account's user and ask if he can identify legit keys.
4. If the keys can not be identified remove all entries from authorized_keys.

## MITRE ATT&CK Information
Adversaries may modify the SSH `authorized_keys` file to maintain persistence on a victim host. Linux distributions and macOS commonly use key-based authentication to secure the authentication process of SSH sessions for remote management. The `authorized_keys` file in SSH specifies the SSH keys that can be used for logging into the user account for which the file is configured. This file is usually found in the user's home directory under `<user-home>/.ssh/authorized_keys`. Users may edit the system’s SSH config file to modify the directives PubkeyAuthentication and RSAAuthentication to the value "yes" to ensure public key and RSA authentication are enabled. The SSH config file is usually located under `/etc/ssh/sshd_config`.

Adversaries may modify SSH `authorized_keys` files directly with scripts or shell commands to add their own adversary-supplied public keys. This ensures that an adversary possessing the corresponding private key may log in as an existing user via SSH.

## General
There are two keys that are used in SSH for authentication. Public key has to be added to `authorized_keys` file to work. User that is connecting need the private key to pass the authntication. It can be specified with `-i` option.
`.ssh` directory permissions must be set to 700, public key `id_rsa.pub` should be 644, `id_rsa` (private key) 700, `authorized_keys` 600.
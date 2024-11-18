### Configure encrypted password using vault

1. Create an `encrypted_string` of your password:
    * $`echo -n 'netapp1234' | ansible-vault encrypt_string --stdin-name password`
    `New Vault password: `          # Enter a decrypt key / token (not the password)
    `Confirm New Vault password: `
    *  Note: To capture to a file, add `> filename.yml`
        * $`echo -n 'netapp1234' | ansible-vault encrypt_string --stdin-name password > password.yml`

2. Copy output, or file contents, into vault.yml file for password entry

3. Add `--vault-id` to ansible command to run it with an encrypted password
    * Example: `ansible-playbook -i inventory/hosts -l na site.yml -e service=svm_initial --vault-id @prompt`
    
4. Provide decrypt key to unlock encrypted string
    * `Vault password (default):`   # Enter the decrypt key / token (not the password) entered above
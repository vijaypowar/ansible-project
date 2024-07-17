### This project includes ansible scripts and directory structure for huge environment of servers.

Strategy for management of multiple Ansible Vault passowrds using selector script.

>Steps

1] Create separate file for each password of environment.

echo 'password_for_vault1' > ~/.ansible_vault_admin_pass
chmod 600 ~/.ansible_vault_admin_pass

echo 'password_for_vault2' > ~/.ansible_vault_user_pass
chmod 600 ~/.ansible_vault_user_pass

2] Create Vault Password Script: It determines which password file to use

nano ~/.ansible_vault_pass_selector.sh
#!/bin/bash

if [[ $1 == "vault1" ]]; then
  cat ~/.ansible_vault_pass_1
elif [[ $1 == "vault2" ]]; then
  cat ~/.ansible_vault_pass_2
else
  echo "Vault ID not recognized" >&2
  exit 1
fi

>Make the script executable:
chmod +x ~/.ansible_vault_pass_selector.sh

3] Configure ansible.cfg file to use the script

nano ansible.cfg
[defaults]
vault_password_file = ~/.ansible_vault_pass_selector.sh

4] Encrypt Data with Multiple Vaults

ansible-vault encrypt_string 'YOUR_SECURE_DATA' --name 'variable_name' --vault-id vault1
ansible-vault encrypt_string 'YOUR_OTHER_SECURE_DATA' --name 'another_variable' --vault-id vault2

Example:

ansible-vault encrypt_string 'admin_password_1' --name 'admin_password' --vault-id vault1
ansible-vault encrypt_string 'admin_password_2' --name 'admin_password' --vault-id vault2


5] Use encrypted variables in your playbooks or roles, specifying the appropriate vault ID where necessary.

6] Example role using encrypted variables

role/user_management/tasks/main.yml

---
- name: Ensure admin user exists
  user:
	name: admin
	state: present
	groups: "sudo"
	append: yes
	create_home: yes
	shell: /bin/bash

- name: Set password for admin user
  user:
	name: admin
	password: "{{ admin_password }}"
  vars:
	admin_password: !vault |
      	$ANSIBLE_VAULT;1.1;AES256
      	vault1_data_for_admin_password
	another_password: !vault |
      	$ANSIBLE_VAULT;1.1;AES256
      	vault2_data_for_user_password

7] Run Playbooks: When you run playbook, the custom script will be called with the vault ID

ansible-playbook playbooks/site_production.yml -i inventory/production/hosts --vault-id vault1@~/.ansible_vault_pass_selector.sh

